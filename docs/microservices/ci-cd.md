---
title: CI/CD-jét mikroszolgáltatások
description: Folyamatos integrációt és mikroszolgáltatások folyamatos szállítási
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 7d8a81b7bc236e50d722a68a0115b9220d4e094f
ms.sourcegitcommit: 786bafefc731245414c3c1510fc21027afe303dc
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/12/2017
ms.locfileid: "26653062"
---
# <a name="designing-microservices-continuous-integration"></a>Mikroszolgáltatások tervezése: folyamatos integrációt

Folyamatos integrációt és folyamatos kézbesítési (CI/CD) a kulcsfontosságú követelmény az mikroszolgáltatások sikeres megvalósításához. Egy jó CI/CD folyamat nélkül, nem érhető el a agilitást, amely mikroszolgáltatások készletéről. A CI/CD kapcsolatban felmerülő kihívások mikroszolgáltatások némelyike erednek több kódbázis és a különböző szolgáltatásokat a heterogén buildkörnyezeteket. Ez a fejezet ismerteti, a kihívás, és azt javasolja, hogy az egyes módszerek a problémára.

![](./images/ci-cd.png)

Gyorsabb kiadás ciklusok a legnagyobb okai a mikroszolgáltatások architektúra elfogadására. 

Egy tisztán egységes alkalmazásban nincs kimenete az alkalmazás futtatható fájlját egy összeállítási folyamat. Minden fejlesztési munkahelyi hírcsatornák be ez az adatcsatorna esetén. Egy magas prioritású hiba található, ha egy javítást kell lehet integrált, tesztelni, és közzé, amely késleltetheti-e a kiadás az új funkciók. IGAZ, hogy ezek a problémák csökkentheti jól faktorált modulokat, és használja a szolgáltatás ágak kódmódosításokat hatás minimalizálása érdekében. De, ha az alkalmazás növekszik összetettebb, és további szolgáltatások lesznek hozzáadva, egy monolit kiadás folyamatát jobban rideg, és valószínűleg törés válnak. 

Következő mikroszolgáltatások alapvetően soha nem kell egy hosszú kiadás vonat, ahol minden csoport rendelkezik sorában lekérdezni. A szolgáltatás "A" módosítások szolgáltatásban való egyesíthetők, "B" várakozás nélkül is megjelenhetnek egy frissítés bármikor, épülő csapat tesztelése és telepítése. A CI/CD folyamat fontos, hogy ez lehetséges. A kiadási folyamat kell automatizált és nagymértékben megbízható, így frissítéseinek telepítéséhez a kockázata, így a lehető legkisebb. Ha naponta vagy naponta több alkalommal felszabadítása vannak üzemi környezetben, regresszió vagy szolgáltatás zavarokat nagyon ritkán fordul elő kell lennie. Egy időben Ha egy hibás frissítés telepítve, rendelkeznie kell gyorsan állíthatja vissza, illetve egy korábbi verzióját a szolgáltatás előregörgetéseket megbízható módot.

![](./images/cicd-monolith.png)

Ha a döntésről bővebben CI/CD, azt vannak valóban van szó több kapcsolódó folyamatokhoz: folyamatos integrációt, a folyamatos és a folyamatos üzembe helyezést.

- Folyamatos integrációt azt jelenti, hogy kódmódosításokat gyakran egyesülnek a főágba automatizált build használatával győződjön meg arról, hogy a főágba kód mindig kiváló minőségben folyamatok tesztelése.

- Folyamatos kézbesítési azt jelenti, hogy a kód módosítására, amelyek megfelelnek a CI-folyamat automatikusan közzétett hasonló környezethez. Az éles környezetbe telepítés manuális jóváhagyásra lehet szükség, de egyébként automatikus. A célja, hogy a kód mindig legyen *készen* való telepítése éles környezetben.

- Folyamatos üzembe helyezés, az azt jelenti, hogy a kód módosítására, amelyek megfelelnek a CI/CD folyamat automatikusan telepítve éles környezetben.

Kontextusában Kubernetes és mikroszolgáltatások létrehozására a CI szakasz aggasztanak létrehozása és tesztelése tároló képek és kérdez le ezeket a képeket tároló beállításjegyzékbeli, hogy. A telepítési fázisban pod specifikációk frissítése a legújabb éles kép átvételéhez.

## <a name="challenges"></a>Kihívásai

- **Sok kisméretű független kódbázis**. Minden team saját szolgáltatása, és a saját build összeállításáért felelős. Egyes szervezeteknél csoportokat használhat külön adattárakban. Ez a helyzet, ahol csoportok között megoszlik hogyan hozhat létre a rendszer ismerete, és a szervezet senki sem tudja, hogy a teljes alkalmazás központi telepítéséről vezethet. Például mi történik, a vész-helyreállítási forgatókönyv, ha szeretné gyorsan üzembe helyezhet egy új fürtre?   

- **Több nyelv és keretrendszer**. Minden egyes csapattal, a saját vegyesen technológiák használatával nehézkes lehet, amely a teljes szervezetben egyetlen build folyamatot létrehozni. A létrehozási folyamat kellően rugalmas ahhoz, hogy minden team módosíthatja azt a nyelvet vagy keretrendszert az általuk választott kell lennie. 

- **Integráció és a betöltés tesztelés**. A saját tempójában frissítések felszabadítása csapatok azt kihívást jelenthet robusztus-végpontok tesztelése, különösen ha szolgáltatások függőségi viszonyban vannak más szolgáltatások tervezéséhez. Ezenkívül egy teljes éles fürt futtató lehet költséges, így nem valószínű, hogy minden team fogja tudni futtatni a saját teljes fürt éles méretezik csak teszteléshez. 

- **Kiadáskezelés**. Minden csoport lehetővé teszi egy frissítés telepítését üzemi környezetben kell rendelkeznie. Ez nem jelenti azt, hogy minden csapattag ehhez engedélyekkel rendelkezik-e. Azonban rendelkezik egy központosított Kiadáskezelő szerepkör csökkentheti a sebesség, a központi telepítések. A további, hogy a CI/CD folyamat automatikus, és megbízható, kevesebb e kell lennie egy központi hatóság szükség. Említett, előfordulhat, hogy eltérő házirendek a fő funkció frissítések kisebb hibák javításával és felszabadítása. Decentralizált alatt nem jelenti azt nulla irányítás kell lennie.

- **Tároló kép versioning**. A fejlesztési és tesztelési ciklus során a CI/CD folyamat sok tároló lemezképet fog létrehozni. Csak a kiadásban a zel olyanokat is, majd közül csak az adott kiadás jelöltek fogja lekérni leküldött éles környezetben. Törölje a jelet versioning stratégiát, megállapításához, hogy mely képek üzemi jelenleg telepítve vannak, és vissza lehet vonni egy korábbi verzióját szükség kell. 

- **Frissítheti a szolgáltatást**. Amikor új verzióra frissít egy szolgáltatás, azt nem szabad törés attól függő más szolgáltatások. Ha így tesz, a működés közbeni frissítés, egy adott időn belül verzióit vegyesen futtatásakor lesz. 
 
Ezek a kihívások alapvető feszültség tükrözik. Másrészről szükséges működjön, a lehető függetlenül. Néhány koordinációs, másrészt van szükség, hogy egy személy teheti a feladatokat, például az integráció a teszt, az új fürtön a teljes megoldás újbóli vagy rossz frissítés visszaállítása. 
 
## <a name="cicd-approaches-for-microservices"></a>CI/CD megközelítés használatos mikroszolgáltatások

Ajánlott minden szolgáltatás-csoport containerize azok összeállító környezetet. Ebben a tárolóban kell rendelkeznie az összes build eszköz a kód összetevők, a szolgáltatás létrehozásához szükséges. Gyakran a nyelv és keretrendszer hivatalos Docker kép is megtalálhatja. Akkor `docker run` vagy a Docker Compose a build futtatásához. 

Ezzel a megközelítéssel nagyon egyszerű, egy új build környezet beállítása. Egy fejlesztő, akinek szeretne készíteni a kód nem kell telepítenie a build eszközöket, de egyszerűen futtatja a tároló kép. Lehet, hogy ennél is fontosabb, a build kiszolgáló beállítható úgy, hogy ugyanazt végzi. Ezzel a módszerrel nem kell ezekhez az eszközökhöz a build-kiszolgálóra telepítse, vagy eszközök ütköző verzióinak kezelése. 

A helyi fejlesztéshez és teszteléshez használja a Docker belül egy tárolót a szolgáltatás futtatásához. Ez a folyamat részeként szükség lehet más tárolókat utánzatait szolgáltatások vagy a helyi tesztelés szükséges teszt adatbázisok futtatásához. Ezekhez a tárolókhoz koordinálja a Docker Compose használatával, vagy helyileg történő futtatása Kubernetes Minikube segítségével. 

Amikor készen áll a kód, nyisson meg egy lekérési kérelmet és egyesítési fő be. Ez feladatot indít el a build kiszolgálón:

1. A kód eszközök felépítéséhez. 
2. A kód egység tesztek futtatásához.
3. A tároló lemezképet létre.
4. Tesztelje a tároló kép működési tesztek futtatása a futó tárolóba. Ez a lépés is tényleges hibákat a Docker-fájlban, például egy hibás belépési ponthoz.
5. Küldje le a lemezképet tároló beállításjegyzékbeli.
6. A tesztfürthöz frissítése az új lemezkép integrációs tesztek futtatásához.

Ha a kép készen éles környezetben, frissítse a telepítési fájlok a legújabb kép, bármely Kubernetes konfigurációs fájlokat adja meg szükség szerint. Az éles fürt követően telepítse a frissítést.

Az alábbiakban néhány javaslatot adja meg a megfelelő központi telepítések további megbízható:
 
- Adja meg a teljes szervezetre kiterjedő egyezmények tároló címkék, versioning és elnevezési szabályai a fürthöz (három munkaállomás-csoporttal, szolgáltatások és így tovább) üzembe helyezett erőforrás. Amely megkönnyítheti a telepítési problémák elemzéséhez. 

- Hozzon létre két külön nyilvántartó, egy fejlesztési/tesztelési és egy üzemi. Lemezkép nem leküldéses a termelési beállításjegyzék, amíg készen áll az éles üzembe helyezés. Ha ez az eljárás egyesíthető szemantikai versioning tároló képek, csökkentheti az esélyét, hogy véletlenül telepítése egy olyan verzióra, a kiadáshoz nem engedélyezett.

## <a name="updating-services"></a>Frissítési szolgáltatások

Nincsenek olyan szolgáltatás, amely már szerepel a termelési frissítési különböző stratégiákat. Itt három közös beállításokat tárgyaljuk: működés közbeni frissítés, kék, zöld telepítési és Kanári kiadás.

### <a name="rolling-update"></a>Működés közbeni frissítés 

A működés közbeni frissítés a szolgáltatás új példányának telepítése, és az új példányok megkezdése kérelmek azonnal. Az új példányok kapja meg, mert a rendszer eltávolítja a korábbi példányok.

Működés közbeni frissítés esetén Kubernetes az alapértelmezett viselkedés, a központi telepítés pod spec frissítésekor. A központi telepítés tartományvezérlő létrehoz egy új ReplicaSet esetében a frissített három munkaállomás-csoporttal. Majd méretezés során az új ReplicaSet le a régit, a kívánt replikaszám fenntartásához méretezés közben. Amíg az új címkék készen áll az nem törli a régi három munkaállomás-csoporttal. Kubernetes tartja a frissítési előzményeket, így kubectl használatával állítsa vissza egy frissítés szükség esetén. 

Ha a szolgáltatás indítása sok feladat hajt végre, megadhat egy készenléti mintavételt. A készenléti mintavételi jelentést készít, megkezdheti a forgalom fogadására a tároló esetén. Kubernetes nem forgalmat küldeni fogyasztanak, amíg a mintavétel sikeresnek jelzi. 

A működés közbeni frissítések az egyik kihívás, hogy a frissítési folyamat alatt vegyesen a régi és új verzióinak futtatását és forgalom fogadására. Ebben az időszakban a kérelem sikerült beolvasása irányítja át a két egyikét. Előfordulhat, hogy vagy nem problémákhoz vezethet, attól függően, hogy a két verziója közötti különbségek hatókör. 

### <a name="blue-green-deployment"></a>Kék-zöld központi telepítés

A kék-zöld telepítésében központi telepítése mellett az előző verzió új verziója. Után az új verzió ellenőrzéséhez át kell váltania az új verzió egyszerre korábbi verziójából származó összes forgalmat. A kapcsoló után figyelheti az alkalmazást a problémákat. Ha valamilyen hiba, a régi verzióról kicserélheti. Ha nem jelez problémákat, törölheti a régi verzióját.

Hagyományos egységes vagy N szintű alkalmazással kék, zöld telepítési általában azt jelentette, hogy két azonos környezet kiépítési. Lenne az új verzió telepítése az átmeneti környezetben való használatra, majd ügyfél forgalmát átirányítja az átmeneti &mdash; például által csere VIP-címek.

A Kubernetes nem kell kiépíteni egy külön fürt kék-zöld központi telepítések elvégzéséhez. Ehelyett kihasználhatja a választók. Hozzon létre egy új központi telepítési erőforrást egy új pod spec és egy másik címkék. Hozza létre a központi telepítés, a korábbi központi telepítés törölhessék vagy módosíthassák a rá mutató szolgáltatás nélkül. Miután az új három munkaállomás-csoporttal futnak, a szolgáltatás választó megfelelően az új központi telepítési frissítheti. 

A kék-zöld központi telepítések előnye, hogy a szolgáltatás minden a három munkaállomás-csoporttal vált egy időben. Után a szolgáltatás frissítve lett, új kérések beolvasása irányítása az új verzióra. Egyik hátránya, hogy a frissítés során futtatja kétszer, számos három munkaállomás-csoporttal a szolgáltatás (az aktuális és a következő). Ha a három munkaállomás-csoporttal van szükség a CPU és memória-erőforrásokat számos, szükség lehet terjessze ki a fürt ideiglenesen az erőforrás-felhasználás kezelésére. 

### <a name="canary-release"></a>Kanári kiadás

Egy Kanári kiadásban megkezdik frissített verzióját egy kis ügyfelek számára. Az új szolgáltatás működését majd mielőtt végrehajtaná az összes ügyfél figyelheti. Ez lehetővé teszi egy lassú bevezetés szabályozott módon tegye, tekintse át az adatok Valóságosak, és a problémák előtt az összes ügyfelet érintenének.

Kanári kiadási nem kezeléséhez kék-zöld vagy a működés közbeni frissítés, mint összetett, mert a szolgáltatás különböző verzióihoz való dinamikusan kell továbbítani kérelmek. Kubernetes konfigurálhatja egy szolgáltatás span két replikakészletekhez (minden verzió egyet), és módosítsa kézzel a replikát számát. Azonban ez a megközelítés ahelyett, hogy coarse-grained, három munkaállomás-csoporttal keresztül kiegyensúlyozza Kubernetes betöltése módjával. Például ha tíz replikák összesen, akkor is csak az eltolás mértékét megadó 10 %-os lépésekben forgalmat. Szolgáltatás rácsvonal használ, ha a szolgáltatás háló útválasztási szabályokat segítségével kifinomultabb Kanári kiadási stratégia megvalósításához. Íme néhány forrás, amelyek segíthetnek:

- Szolgáltatás háló nélkül Kubernetes: [Kanári központi telepítések](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#canary-deployments)
- Linkerd: [irányítása dinamikus](https://linkerd.io/features/routing/)
- Istio: [Kanári üzembe helyezése Istio](https://istio.io/blog/canary-deployments-using-istio.html)

## <a name="conclusion"></a>Összegzés

Az elmúlt években egy tengeri változás történt az iparág, egy mozdulattal épület *rekord rendszerek* az épület *engagement rendszerek*.

Rekord rendszereket hagyományos vissza office-kezelési alkalmazások. Ezek a rendszerek középpontjában van gyakran helyezkedik el, amely a igazság egyetlen forrása egy RDBMS. A kifejezés "engagement rendszer" követel Geoffrey Moore annak 2011 dokumentum *rendszerek az Engagement és a jövőben vállalati informatikai*. Bevonási rendszereket alkalmazások kommunikáció és az együttműködési összpontosít. Valós idejű személyek szolgáltatáshoz csatlakozik. Rendelkezésre álló 24/7 kell lenniük. Új szolgáltatások rendszeresen új, anélkül, hogy az alkalmazás offline állapotba helyezése. Felhasználók várhatóan több és kevesebb türelmet nem várt késedelmeket vagy leállás.

A fogyasztó címtérben jobb felhasználói élményt mérhető üzleti értéke lehet. Ennyi idő alatt, amely a felhasználó kapcsolatba lép egy alkalmazással is lefordítása közvetlenül bevétel. És abban a tartományban, üzleti rendszerek, felhasználók elvárásainak megváltoztak. Ha ezek a rendszerek célja a kommunikáció és az együttműködés elősegítése, akkor szükséges, azok a felhasználók felé néző alkalmazások köteg.

Mikroszolgáltatások a változó fekvő választ. Decomposing egy egységes alkalmazás lazán összekapcsolt szolgáltatások egy csoporthoz, azt is szabályozhatja a kiadási ciklus minden szolgáltatás, és állásidő vagy megtörje a módosítások nélkül gyakori frissítések engedélyezése. Mikroszolgáltatások létrehozására is hozzájárulhat a méretezhetőséget, hiba elkülönítési és rugalmasság. Eközben felhőplatformokkal van így könnyebben létrehozása és futtatása mikroszolgáltatások létrehozására, az automatizált üzembe helyezést számítási erőforrásokat, a tároló orchestrators egy szolgáltatás, valamint eseményvezérelt kiszolgáló nélküli környezetekben.

De azt látja, mint mikroszolgáltatások architektúrák lehetőség van-e nagy mennyiségű kihívást. A sikeres, kell elindítania teli kialakítást. Meg kell helyezni, gondosan meg kell hogy elemzése a tartományhoz, technológiák kiválasztása, adatok modellezését, API-k tervezésekor, és egy összetett DevOps kulturális környezet létrehozása. Reméljük, hogy ez az útmutató, és az ahhoz mellékelt [megvalósítása hivatkozhat](https://github.com/mspnp/microservices-reference-implementation), az út megvilágítására hozzájárult. 

