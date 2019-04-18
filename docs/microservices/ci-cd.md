---
title: CI/CD a mikroszolgáltatásokhoz
description: Folyamatos integráció és folyamatos készregyártás a mikroszolgáltatásokat.
author: MikeWasson
ms.date: 03/27/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: c52ff3d0a330f564e5f7e9b0b07f0ba84c328c8b
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639784"
---
# <a name="cicd-for-microservices-architectures"></a>CI/CD-mikroszolgáltatás-architektúrákat

Gyorsabb kiadási ciklusokhoz főbb előnyei a mikroszolgáltatás-architektúrákat tartoznak. Azonban egy jó CI/CD-folyamat nélkül nem érhető el a rugalmasságot, mellyel a mikroszolgáltatások garantálniuk. Ez a cikk a kihívásokat ismerteti, és javasolja a probléma néhány olyan módszert.

## <a name="what-is-cicd"></a>Mit jelent a CI/CD-ről?

Amikor CI/CD az, hogy valóban több kapcsolódó folyamatok beszélünk: Folyamatos integráció, a folyamatos teljesítés és a folyamatos üzembe helyezés.

- **Folyamatos integráció**. Kód gyakran összefésülés megtörténne a fő ágat. Automatikus buildelési és tesztelési folyamatokat győződjön meg arról, hogy kódot a fő ág mindig gyártási minőségű.

- **A folyamatos teljesítés**. Adja át a CI folyamat kódváltozások a rendszer automatikusan közzéteszi a utolsóig üzemi környezetben. Az élő éles környezetben való üzembe helyezés manuális jóváhagyásra lehet szükség, de egyébként automatizált. Az a célja, hogy a kód mindig legyen *készen* helyezheti éles üzembe.

- **Folyamatos üzembe helyezés**. Kód módosítja, hogy a rendszer automatikusan telepíti az előző két lépést pass *éles környezetbe*.

Az alábbiakban néhány célja a mikroszolgáltatási architektúra egy robusztus CI/CD folyamatot:

- Minden egyes csapat létrehozhat és üzembe helyezése a szolgáltatások egymástól függetlenül, tulajdonos érintő vagy más csapatok megszakítása nélkül.

- A szolgáltatás egy új verziója éles üzembe helyezés, mielőtt megtörténik fejlesztési/tesztelési/QA Environment ellenőrzés céljából. Minőségi kapuk a rendszer minden egyes fázisában érvényesíti.

- A szolgáltatás egy új verziója is telepíthető az előző verzió párhuzamosan lesz.

- Megfelelő hozzáférés-vezérlési házirendeket vannak érvényben.

- A tárolóalapú számítási feladatokhoz megbízható a tárolórendszerképeket az éles környezetben üzembe helyezett.

## <a name="why-a-robust-cicd-pipeline-matters"></a>Miért fontos a egy robusztus CI/CD-folyamat

A hagyományos monolitikus alkalmazások esetén nincs egy egyetlen build folyamatot, amelynek a kimenete az alkalmazás futtatható. Minden fejlesztési munka hírcsatornák a folyamatba. Ha egy magas prioritású hiba található, a javítás kell lennie integrált, tesztelését és kiadását, amely késleltetheti-e új funkciók megjelenésével. Jól faktorált modulokat, és a szolgáltatás ágak használatával kódmódosításokat minimalizálása érdekében csökkentheti ezeket a problémákat. Azonban, ha az alkalmazás összetettebb növekszik, és további szolgáltatások lesznek hozzáadva, mikroszolgáltatásokká kiadási folyamata általában törékennyé teszi és valószínűleg érvényteleníteni válik.

A mikroszolgáltatások filozófia a következő soha nem kell egy hosszú kibocsátási train, ahol minden egyes csapat beolvasása a sor van. "A" módosításait a szolgáltatás részeként történik, hogy a "B" várakozás nélkül is megjelenhetnek egy tetszőleges időpontban, frissítési szolgáltatás hangolt tesztelését és üzembe helyezve.

![CI/CD mikroszolgáltatásokká ábrája](./images/cicd-monolith.png)

A kiadások gyors megjelenését eléréséhez a kibocsátási folyamat automatizált és a kockázat minimalizálása érdekében magas megbízhatóságú kell lennie. Ha felszabadít éles naponta vagy naponta többször, regressziót vagy szolgáltatáskimaradás nagyon ritkán kell lennie. Egyszerre egy rossz frissítés üzembe helyezése, ha rendelkeznie kell egy megbízható módszerre gyorsan visszaállni vagy előregörgethet egy korábbi verzióját, a szolgáltatás.

## <a name="challenges"></a>Problémák

- **Sok kis független kódbázissal**. Minden egyes csapat felelős a saját szolgáltatás, amelynek a saját build folyamat létrehozásához. Egyes szervezetekben a csapat a előfordulhat, hogy külön kódtárházak. Olyan helyzet, amelyben az ismeretek, hogyan hozhat létre a rendszer a csapatok között megoszlik, és a szervezet senki sem tudja, hogy a teljes alkalmazás központi telepítése különálló tárházak vezethet. Például mi történik, a vész-helyreállítási helyzetekre, ha gyorsan üzembe helyezhet egy új fürtre van szüksége?

    **Kockázatcsökkentési**: Rendelkezik egy egységes és automatikus készíthet és helyezhet üzembe szolgáltatásokat, így ezt a tudást nem "rejtett" belül minden egyes csapat.

- **Több nyelv és keretrendszer**. Az egyes csapata a saját technológiák vegyítése használatával nehézkes lehet a szervezet egészében működő egyetlen buildelési folyamat létrehozásához. A létrehozási folyamat kellően rugalmas ahhoz, hogy minden egyes csapat tesztkörnyezetéhez, az általuk választott nyelvet vagy keretrendszert kell lennie.

    **Kockázatcsökkentési**: A létrehozási folyamat minden egyes szolgáltatás tárolóba. Ezzel a módszerrel az összeállítási rendszer ugyanúgy képesnek kell lennie a tárolók futtatásához.

- **Integráció és a load testing**. Csapatok számára a saját tempójában frissítéseket ad ki ez kihívást jelenthet robusztus – teljes körű tesztelés, különösen ha szolgáltatások függőségekkel rendelkezik más szolgáltatások tervezéséhez. Ezen felül egy teljes éles fürtöt futtat költséges lehet, ezért nem valószínű, hogy minden egyes csapat a saját teljes fürtöt fog futni éles skálázását követve rugalmasan méretezhető, csak teszteléshez.

- **Kiadáskezelés**. Minden egyes csapat egy frissítés telepítése éles képesnek kell lennie. Ez nem jelenti azt, hogy minden csapattag engedélyekkel rendelkezik-e. Azonban egy központosított Kiadáskezelő szerepkör kellene csökkentheti a központi telepítések adatról van szó.

    **Kockázatcsökkentési**: A további, hogy a CI/CD-folyamat automatikus, és megbízható, a kevésbé ott kell lennie egy központi hatóság szüksége. Ugyanakkor előfordulhat, hogy kisebb hibajavítások és a főbb funkciókat frissítések kiadása különböző házirendeket. Folyamatban decentralizált nulla cégirányítási nem jelenti azt.

- **Szolgáltatási hírek**. Amikor új verzióra frissít egy szolgáltatás, azt más a tőle függő szolgáltatások nem szünet.

    **Kockázatcsökkentési**: Központi telepítési módszerek, például a kék-zöld vagy canary kiadás használata a nem kompatibilitástörő változások. Parancsban történt használhatatlanná API-módosítás, párhuzamosan lesz az előző verzió az új verzió telepítése. Ezzel a módszerrel szolgáltatásokat, amelyek az előző API felhasználására is frissíthetők és az új API tesztelése. Lásd: [szolgáltatások frissítése](#updating-services), az alábbi.

## <a name="monorepo-vs-multi-repo"></a>Monorepo vs több adattár

Mielőtt létrehozná a CI/CD munkafolyamat, ismernie kell a hogyan kódbázis fog strukturált és felügyelt.

- Működnek-e a teams, külön tárházakban vagy egy monorepo (egyetlen adattár)?
- Mi az az elágazási stratégia?
- Akik beküldéssel telepíthet az éles környezetbe? Egy kiadási szerepkör van?

A monorepo megközelítés hozd hónappal számolunk, de vannak előnyei és hátrányai is.

| &nbsp; | Monorepo | Több adattárakkal |
|--------|----------|----------------|
| **Előnyök** | Kód megosztása<br/>Könnyebben szabványosíthatja a kódot, és azokat az eszközöket<br/>Könnyebben újrabontása kód<br/>Könnyebben – a szabályzat egyetlen nézetben<br/> | Egyértelmű tulajdonjog csapatonként<br/>Potenciálisan kevesebb ütközések egyesítése<br/>Elválasztás álló kényszerítése segít |
| **Problémák** | Megosztott programkód módosítása hatással lehet a több mikroszolgáltatások<br/>Az egyesítési ütközések nagyobb lehetőségeit<br/>Azokat az eszközöket kell méretezhető, nagy méretű kódbázis<br/>Hozzáférés-vezérlés<br/>Összetettebb telepítési folyamat | Nehezebb lehet megosztani a kódot<br/>Nehezebb kódolási szabványok kikényszerítésére<br/>Függőségkezelés<br/>Kód alap, a gyenge felderíthetőség szórt<br/>Megosztott infrastruktúra hiánya

## <a name="updating-services"></a>Szolgáltatás frissítése

Számos különféle stratégiát, amely már éles környezetben a szolgáltatás frissítése. Itt három általános beállítások tárgyaljuk: A működés közbeni frissítés, a kék-zöld üzembe helyezés és a canary kiadás.

### <a name="rolling-updates"></a>A működés közbeni frissítés

A működés közbeni frissítés a szolgáltatás új példányok üzembe helyezésekor, és az új példányok, indítsa el azonnal a kérelmek fogadására. Az új példányok merülnek fel, mint az előző példányok törlődnek.

**Példa:** Kubernetes, a működés közbeni frissítések alapértelmezés szerint akkor, ha frissíti a pod specifikációja üzembe helyezéséhez. A központi telepítési vezérlő hoz létre egy új ReplicaSet a frissített podok számára. Ezután a méretezés során az új ReplicaSet vertikális leskálázást a régit, a kívánt replikáinak száma fenntartása közben. Régi podok azzal nem törli, amíg készen áll az újakat. Kubernetes előzményeket a frissítést, így állíthat vissza egy frissítést szükség esetén.

Egy igazi kihívást a működés közbeni frissítés, hogy a frissítési folyamat alatt vegyesen a régi és új verziókat futtató és forgalmat fogadó. Ebben az időszakban bármilyen kérést sikerült irányítva a két verzió valamelyikéhez.

Kompatibilitástörő API-módosítás, bevált gyakorlat, hogy mindkét verziót egymás mellett támogatja, mindaddig, amíg az előző változat összes ügyfelei már frissítettek. Lásd: [API-k verziókezelése](./design/api-design.md#api-versioning).

### <a name="blue-green-deployment"></a>Kék-zöld üzembe helyezés

A kék-zöld üzembe helyezés helyezi üzembe az új verziót az előző verzió mellett. Ellenőrizze, hogy az új verziót, akkor váltson egyszerre az előző verzióhoz képest az új verzió minden forgalmat. A váltás után az alkalmazás kapcsolatos problémákat figyeli. Ha hiba lép fel, kicserélheti az a korábbi verzióra. Ha nem talált nincs probléma, törölheti a régi verzióját.

Egy hagyományos monolitikus vagy N szintű alkalmazáshoz a kék-zöld üzembe helyezés általában kizárólag két azonos környezet kiépítése. Szeretné az új verzió üzembe helyezése átmeneti környezetben, akkor az átmeneti ügyfél forgalom átirányítása &mdash; például virtuális IP-CÍMEK felcserélésével címek. A mikroszolgáltatási architektúrákban frissítések fordulhat elő, a mikroszolgáltatás-szintjén, így általában szeretné ugyanabban a környezetben helyezze üzembe a frissítést, és felcserélni service discovery mechanizmus használata.

**Példa**. A Kubernetes nem kell tennie a kék-zöld üzembe önálló fürt üzembe helyezése. Ehelyett kihasználhatja a választók. Hozzon létre egy új központi telepítési erőforrás egy új pod specifikációja és címkék külön készletét. Hozza létre a központi telepítés nélkül a korábbi központi telepítés törlését vagy módosítását a szolgáltatás, amely azt mutat. Miután az új podok is futnak, frissítheti a szolgáltatás választóra felel meg az új központi telepítés.

Egy kék-zöld üzembe helyezés hátránya, hogy a frissítés során futtatja kétszer, számos podok a szolgáltatáshoz (az aktuális és a következő). Ha a podok sok CPU és memória-erőforrások, szükség lehet horizontális felskálázása a fürt ideiglenesen az erőforrás-használat kezelésére.

### <a name="canary-release"></a>Canary kiadás

Canary kiadás, a bevezetés során fokozatosan egy frissített verziót egy kis mennyiségű ügyfelet. Az új service viselkedését majd mielőtt megvalósítaná minden ügyfélnek figyeli. Ez lehetővé teszi egy lassú bevezetés, mégpedig szabályozott módon tegye, tekintse át az adatokon, és helyszíni problémák előtt az összes érintett felhasználókat.

Canary kiadás meg kell dinamikusan irányíthatja a kérelmeket a szolgáltatás különböző verzióit azért bonyolultabb, mint kék-zöld vagy a működés közbeni frissítés kezeléséhez.

**Példa**. A Kubernetes konfigurálhatja egy szolgáltatás span két replikakészletekhez (egy minden verzió esetében) és a replika számát manuálisan módosíthatja. Azonban ez a megközelítés inkább coarse-grained, Kubernetes betöltése módja miatt elosztja a podok között. Például ha 10 replika összesen, akkor is csak áthelyezni a forgalmat 10 %-os lépésekben. Egy szolgáltatás háló használja, ha a szolgáltatás háló útválasztási szabályok segítségével canary kiadás ennél kifinomultabb stratégiát megvalósítása.

## <a name="next-steps"></a>További lépések

Ismerje meg, mikroszolgáltatások, Kubernetes futó adott CI/CD-eljárások.

- [CI/CD-mikroszolgáltatások a kubernetes használatával](./ci-cd-kubernetes.md)