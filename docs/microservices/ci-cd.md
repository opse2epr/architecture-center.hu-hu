---
title: CI/CD-mikroszolgáltatások
description: Folyamatos integráció és folyamatos készregyártás a mikroszolgáltatásokat.
author: MikeWasson
ms.date: 10/23/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 1a927beb23a2e45b509b2648a6bccce4c2ad242d
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/08/2019
ms.locfileid: "59068973"
---
# <a name="designing-microservices-continuous-integration"></a>Mikroszolgáltatások tervezése: Folyamatos integráció

Folyamatos integráció és folyamatos készregyártás (CI/CD) egy kulcsfontosságú követelmény az sikeres a mikroszolgáltatás-alapú eléréséhez. Egy jó CI/CD-folyamat nélkül, nem éri el a rugalmasságot, mellyel a mikroszolgáltatások garantálniuk. Néhány a mikroszolgáltatások a CI/CD kapcsolatban felmerülő kihívások merülnek fel, több kódbázissal és életciklusainak buildkörnyezeteket, a különböző szolgáltatások esetében nem. Ez a cikk a kihívásokat ismerteti, és javasolja a probléma néhány olyan módszert.

![CI/CD a mikroszolgáltatásokat ábrája](./images/ci-cd.png)

Gyorsabb kiadási ciklusokhoz a legnagyobb okai a mikroszolgáltatási architektúra elfogadására.

A tisztán monolitikus alkalmazások esetén nincs egy egyetlen build folyamatot, amelynek a kimenete az alkalmazás futtatható. Minden fejlesztési munka hírcsatornák a folyamatba. Ha egy magas prioritású hiba található, a javítás kell lennie integrált, tesztelését és kiadását, amely késleltetheti-e új funkciók megjelenésével. IGAZ, hogy ezek a problémák csökkentheti jól faktorált modulokat, és a szolgáltatás ágak használatával kódmódosításokat minimalizálása érdekében. Azonban, ha az alkalmazás összetettebb növekszik, és további szolgáltatások lesznek hozzáadva, mikroszolgáltatásokká kiadási folyamata általában törékennyé teszi és valószínűleg érvényteleníteni válik.

A mikroszolgáltatások filozófia a következő soha nem kell egy hosszú kibocsátási train, ahol minden egyes csapat beolvasása a sor van. "A" módosításait a szolgáltatás részeként történik, hogy a "B" várakozás nélkül is megjelenhetnek egy tetszőleges időpontban, frissítési szolgáltatás hangolt tesztelését és üzembe helyezve. A CI/CD-folyamat fontos a megoldáson. Úgy, hogy a frissítések telepítése a kockázatát, így a lehető legkisebb a kibocsátási folyamat automatizált és nagy megbízhatóságú, kell lennie. Ha naponta vagy naponta több alkalommal felszabadítása vannak az éles környezetbe, regressziót vagy szolgáltatáskimaradás nagyon ritkán kell lennie. Egyszerre egy rossz frissítés üzembe helyezése, ha rendelkeznie kell egy megbízható módszerre gyorsan visszaállni vagy előregörgethet egy korábbi verzióját, a szolgáltatás.

![CI/CD mikroszolgáltatásokká ábrája](./images/cicd-monolith.png)

Amikor a CI/CD, valóban beszélünk számos kapcsolódó folyamatok: Folyamatos integráció, a folyamatos teljesítés és a folyamatos üzembe helyezés.

- Folyamatos integráció, hogy a kód módosítása gyakran egyesítve a fő ágat, automatizált összeállítási, és győződjön meg arról, hogy kódot a fő ág mindig gyártási minőségű folyamatok tesztelése jelenti.

- Folyamatos készregyártás, az azt jelenti, hogy a kód módosítása, hogy a CI folyamat automatikusan közzéteszi az utolsóig üzemi környezetben. Az élő éles környezetben való üzembe helyezés manuális jóváhagyásra lehet szükség, de egyébként automatizált. Az a célja, hogy a kód mindig legyen *készen* helyezheti éles üzembe.

- Folyamatos üzembe helyezés azt jelenti, kódmódosítás, hogy a CI/CD-folyamat automatikusan telepített éles környezetbe.

Kubernetes- és mikroszolgáltatás-alapú környezetében a Konfigurációelem fázis üzemidejére létrehozása és tesztelése a tárolórendszerképek és ezen rendszerképek leküldése egy tároló-beállításjegyzéket. A központi telepítési fázisban a pod vonatkozó műszaki adatok frissülnek, hogy folytattuk a munkát a legújabb termelési rendszerképet.

## <a name="challenges"></a>Problémák

- **Sok kis független kódbázissal**. Minden egyes csapat felelős a saját szolgáltatás, amelynek a saját build folyamat létrehozásához. Egyes szervezetekben a csapat a előfordulhat, hogy külön kódtárházak. Ez a helyzet, ahol az ismeretek, hogyan hozhat létre a rendszer a csapatok között megoszlik, és a szervezet senki sem tudja, hogy a teljes alkalmazás üzembe helyezése vezethet. Például mi történik, a vész-helyreállítási helyzetekre, ha gyorsan üzembe helyezhet egy új fürtre van szüksége?

- **Több nyelv és keretrendszer**. Az egyes csapata a saját technológiák vegyítése használatával nehézkes lehet a szervezet egészében működő egyetlen buildelési folyamat létrehozásához. A létrehozási folyamat kellően rugalmas ahhoz, hogy minden egyes csapat tesztkörnyezetéhez, az általuk választott nyelvet vagy keretrendszert kell lennie.

- **Integráció és a load testing**. Csapatok számára a saját tempójában frissítéseket ad ki ez kihívást jelenthet robusztus – teljes körű tesztelés, különösen ha szolgáltatások függőségekkel rendelkezik más szolgáltatások tervezéséhez. Ezen felül egy teljes éles fürtöt futtat költséges lehet, ezért nem valószínű, hogy minden egyes csapat fogja tudni a saját teljes fürtöt futtat, éles skálázását követve rugalmasan méretezhető, csak teszteléshez.

- **Kiadáskezelés**. Minden team kell az éles környezetbe helyezheti üzembe az alkalmazásokat egy frissítést. Ez nem jelenti azt, hogy minden csapattag engedélyekkel rendelkezik-e. Azonban egy központosított Kiadáskezelő szerepkör kellene csökkentheti a központi telepítések adatról van szó. A további, hogy a CI/CD-folyamat automatikus, és megbízható, a kevésbé ott kell lennie egy központi hatóság szüksége. Ugyanakkor előfordulhat, hogy kisebb hibajavítások és a főbb funkciókat frissítések kiadása különböző házirendeket. Folyamatban decentralizált jelenti azt, hogy nulla cégirányítási kell lennie.

- **Tároló-lemezkép versioning**. A fejlesztési és tesztelési ciklus során a CI/CD-folyamat számos tárolórendszerképek fog létrehozni. Csak a kiadásban a deduplikációra olyanokat is, majd csak néhányat ezen kiadás jelöltek fog első leküldött éles környezetbe. Egy világos verziókezelési stratégiát kell, hogy tudja, melyik képek az éles környezetben jelenleg telepítve vannak, és vissza lehet vonni egy korábbi verziót, ha szükséges.

- **Szolgáltatási hírek**. Amikor új verzióra frissít egy szolgáltatás, azt más a tőle függő szolgáltatások nem szünet. Ha így tesz, a működés közbeni frissítés, bizonyos idő különböző verzióit vegyesen futtatásakor lesz.

Ezek a kihívások alapvető feszültség tükrözik. Csapatok, másrészt kell működjön, egymástól függetlenül, amennyire csak lehetséges. Néhány koordinálása, másrészt van szükség, hogy egyetlen személy teheti a feladatokat, mint egy integrációs teszt futtatása, újbóli üzembe helyezés a teljes megoldás egy új fürtre vagy egy rossz frissítés visszaállítása.

## <a name="cicd-approaches-for-microservices"></a>A mikroszolgáltatásokat megközelíti a CI/CD

Ajánlott minden service csapata igény szerint tárolóalapúvá alakíthatja az összeállító környezetet. Ez a tároló az összes szükséges hozhat létre a szolgáltatásukhoz kód összetevők build eszköz rendelkezhet. Gyakran a nyelv és keretrendszer hivatalos Docker-rendszerképet is megtalálhatja. Ezt követően használhatja `docker run` vagy a Docker Compose, a build futtatásához.

Ezzel a módszerrel nagyon egyszerű, egy új build-környezetet. Fejlesztőként szeretne a kód felépítéséhez nem kell telepítenie a build-eszközöket, de egyszerűen futtatja a tároló rendszerképét. Talán még fontosabb, a build kiszolgáló beállítható úgy, a végre ugyanezt. Ezzel a módszerrel, nem kell telepíteni ezeket az eszközöket a build-kiszolgálóba, vagy az eszközök ütköző verziók kezelése.

Helyi fejlesztési és tesztelési a Docker használatával futtatni a szolgáltatást a tárolókon belül. Ez a folyamat részeként szükség lehet más utánzatként funkcionáló szolgáltatások vagy helyi teszteléshez szükséges tesztelési adatbázisok rendelkező tárolók futtatásához. Ezek a tárolók összehangolása a Docker Compose használatával, vagy helyileg történő futtatása a Kubernetes Minikube használatával.

Amikor készen áll a kód, nyisson meg egy pull-kérelem és egyesítés főágba be. Ez feladatot indít el a build-kiszolgálón:

1. A kód eszközöket hozhat létre.
2. Egység tesztek futtatásához a kódot.
3. A tárolólemezkép létrehozásához.
4. Ellenőrizze a tároló rendszerképét a funkcionális tesztek futtatása a futó tárolót. Ebben a lépésben a Docker-fájl, például a rossz belépési pont hibáinak használhatják fel őket.
5. A rendszerkép leküldése egy tároló-beállításjegyzéket.
6. A test-fürt frissítése az új lemezkép integrációs tesztek futtatásához.

Amikor a lemezkép készen áll, éles környezetben, frissítse az üzembe helyezési fájlokat a legújabb rendszerképet, beleértve a Kubernetes konfigurációs fájlok megadásához szükség szerint. Ezután alkalmazza a frissítést az éles fürtöt.

Az alábbiakban néhány javaslat megbízhatóbb végzett központi telepítések:

- Adja meg a szervezeti szintű szabályai tároló címkék, verziószámozása és a fürthöz (podok, szolgáltatások és így tovább) üzembe helyezett erőforrások elnevezési szabályainak. Amely megkönnyítheti a üzembe helyezési problémák diagnosztizálásához.

- Hozzon létre két külön tároló-beállításjegyzékek, egyet a fejlesztési-tesztelési és éles környezetben. Nem rendszerkép leküldése a termelési beállításjegyzék, amíg készen áll az éles üzembe helyezés. Ha ez az eljárás kombinálva a tárolórendszerképek Szemantikus verziószámozást, csökkentheti az esélye, hogy véletlenül üzembe helyezése egy olyanra, amely a kiadás nem lett jóváhagyva.

## <a name="updating-services"></a>Szolgáltatás frissítése

Számos különféle stratégiát, amely már éles környezetben a szolgáltatás frissítése. Itt három általános beállítások tárgyaljuk: A működés közbeni frissítés, a kék-zöld üzembe helyezés és a canary kiadás.

### <a name="rolling-update"></a>A működés közbeni frissítés

A működés közbeni frissítés a szolgáltatás új példányok üzembe helyezésekor, és az új példányok, indítsa el azonnal a kérelmek fogadására. Az új példányok merülnek fel, mint az előző példányok törlődnek.

Működés közbeni frissítései az alapértelmezett viselkedést, a Kubernetes üzembe helyezéséhez a pod specifikációja frissítésekor. A központi telepítési vezérlő hoz létre egy új ReplicaSet a frissített podok számára. Ezután a méretezés során az új ReplicaSet vertikális leskálázást a régit, a kívánt replikáinak száma fenntartása közben. Régi podok azzal nem törli, amíg készen áll az újakat. Kubernetes előzményeket a frissítést, így a kubectl használatával visszavonhat egy frissítést szükség esetén.

Ha a szolgáltatás egy hosszú indítási feladat végez, meghatározhatja egy készenlét. A végrehajtandó készenléti teszt jelenti a forgalmat fogadó megkezdheti a tároló esetén. Kubernetes nem küldhetnek forgalmat a pod, amíg a mintavétel sikeres befejezést jelent.

Egy igazi kihívást a működés közbeni frissítés, hogy a frissítési folyamat alatt vegyesen a régi és új verziókat futtató és forgalmat fogadó. Ebben az időszakban bármilyen kérést sikerült irányítva a két verzió valamelyikéhez. Előfordulhat, hogy, amely, vagy nem problémát okozhatnak, a két verziója közötti különbségek függően.

### <a name="blue-green-deployment"></a>Kék-zöld üzembe helyezés

A kék-zöld üzembe helyezés helyezi üzembe az új verziót az előző verzió mellett. Ellenőrizze, hogy az új verziót, akkor váltson egyszerre az előző verzióhoz képest az új verzió minden forgalmat. A váltás után az alkalmazás kapcsolatos problémákat figyeli. Ha hiba lép fel, kicserélheti az a korábbi verzióra. Ha nem talált nincs probléma, törölheti a régi verzióját.

Egy hagyományos monolitikus vagy N szintű alkalmazáshoz a kék-zöld üzembe helyezés általában kizárólag két azonos környezet kiépítése. Szeretné az új verzió üzembe helyezése átmeneti környezetben, akkor az átmeneti ügyfél forgalom átirányítása &mdash; például virtuális IP-CÍMEK felcserélésével címek.

A Kubernetes nem kell tennie a kék-zöld üzembe önálló fürt üzembe helyezése. Ehelyett kihasználhatja a választók. Hozzon létre egy új központi telepítési erőforrás egy új pod specifikációja és címkék külön készletét. Hozza létre a központi telepítés nélkül a korábbi központi telepítés törlését vagy módosítását a szolgáltatás, amely azt mutat. Miután az új podok is futnak, frissítheti a szolgáltatás választóra felel meg az új központi telepítés.

Kék-zöld üzembe előnye, hogy a szolgáltatás minden podok átvált egy időben. A szolgáltatás frissítése után minden új kérés van irányítva az új verzióra. Egyik hátránya, hogy a frissítés során futtatja kétszer, számos podok a szolgáltatáshoz (az aktuális és a következő). Ha a podok sok CPU és memória-erőforrások, szükség lehet horizontális felskálázása a fürt ideiglenesen az erőforrás-használat kezelésére.

### <a name="canary-release"></a>Canary kiadás

Canary kiadás, a bevezetés során fokozatosan egy frissített verziót egy kis mennyiségű ügyfelet. Az új service viselkedését majd mielőtt megvalósítaná minden ügyfélnek figyeli. Ez lehetővé teszi egy lassú bevezetés, mégpedig szabályozott módon tegye, tekintse át az adatokon, és helyszíni problémák előtt az összes érintett felhasználókat.

Canary kiadás meg kell dinamikusan irányíthatja a kérelmeket a szolgáltatás különböző verzióit azért bonyolultabb, mint kék-zöld vagy a működés közbeni frissítés kezeléséhez. A Kubernetes konfigurálhatja egy szolgáltatás span két replikakészletekhez (egy minden verzió esetében) és a replika számát manuálisan módosíthatja. Azonban ez a megközelítés inkább coarse-grained, Kubernetes betöltése módja miatt elosztja a podok között. Például ha tíz replikák összesen, akkor is csak áthelyezni a forgalmat 10 %-os lépésekben. Egy szolgáltatás háló használja, ha a szolgáltatás háló útválasztási szabályok segítségével canary kiadás ennél kifinomultabb stratégiát megvalósítása. Az alábbiakban néhány forrásanyagot, amelyek hasznosak lehetnek:

- Kubernetes szolgáltatás háló nélkül: [Canary központi telepítések](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#canary-deployments)
- Linkerd: [A dinamikus alkalmazáskérelmek irányítása](https://linkerd.io/features/routing/)
- Istio: [Canary központi telepítések Istio használatával](https://istio.io/blog/canary-deployments-using-istio.html)

## <a name="conclusion"></a>Összegzés

Az utóbbi években, egy tenger változás történt az iparág létrehozását egy adatátviteli *rekord rendszerek* az épület *együttműködés rendszerek*.

Rekord rendszereket hagyományos háttérrendszerhez adatkezelési alkalmazások. Ezek a rendszerek középpontjában van gyakran helyezkedik el egy relációsadatbázis-kezelő rendszer, amely az egyetlen hiteles forrásaként. A 2011 tanulmányában "engagement rendszer" kifejezés igényért az Geoffrey Moore, *rendszerek az Engagement és a jövőben nagyvállalati informatikai*. Együttműködés rendszereket összpontosított kommunikációs és együttműködési alkalmazásokat. Valós idejű személyek csatlakoznak. Rendelkezésre álló 24/7 kell lenniük. Új funkciók jelennek meg rendszeresen, az alkalmazás offline állapotba helyezése nélkül. Felhasználók várhatóan további és kevesebb nem várt késedelem és leállás, kis türelmet.

A felhasználói térben jobb felhasználói élményt rendelkezhet mérhető üzleti értéket. Az, hogy mennyi ideig felhasználó folytat egy alkalmazással a előfordulhat, hogy közvetlenül a bevétel fordítja le. És abban a tartományban, üzleti rendszereket, a felhasználói elvárásoknak megváltoztak. Ezek a rendszerek az a célja, hogy a kommunikációs és együttműködési alakíthatott ki, ha azok a felhasználók felé néző alkalmazások köteg kell végrehajtaniuk.

Mikroszolgáltatások a változó fekvő választ. Által decomposing a monolitikus alkalmazásokat felbonthatja lazán kapcsolódó szolgáltatásokra, azt is szabályozhatja az egyes szolgáltatásokat a kiadási ciklus, és lehetővé teszi a gyakori frissítések nélkül állásidő és a használhatatlanná tévő változásai. Mikroszolgáltatások a méretezhetőség, a hiba elkülönítési és a rugalmasság is segítenek. Eközben felhőplatformjain vannak így könnyebben hozhat létre és futtassa a mikroszolgáltatások, a számítási erőforrások automatikus üzembe helyezést, a szolgáltatás, valamint az eseményvezérelt, kiszolgáló nélküli környezetben tárolóvezénylőt.

De mivel úgy tapasztaltuk, kihívások rengeteg lehetőség van mikroszolgáltatás-architektúrákat. A művelet sikeres elvégzéséhez kell elindítania egy alapos terv. A tartomány elemzése, technológiák kiválasztása, modellezési adatok, API-k tervezése és készítése egy fejlett DevOps-kultúra tanulmányozásakor kell tenni. Reméljük, hogy ez az útmutató és a kísérő [megvalósítási hivatkozhat](https://github.com/mspnp/microservices-reference-implementation/tree/v0.1.0-orig), a szállítás megvilágítására segített.
