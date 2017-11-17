---
title: "Mikroszolgáltatások architektúra stílus"
description: "Előnyeit, kihívást és mikroszolgáltatások architektúrák ajánlott eljárásai ismerteti az Azure-on"
author: MikeWasson
ms.openlocfilehash: 6426b3342a319832baf5eec35e9c783ba9348bdd
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="microservices-architecture-style"></a>Mikroszolgáltatások architektúra stílus

Egy mikroszolgáltatások architektúra kicsi, önálló szolgáltatások gyűjteményei. Minden szolgáltatás önálló, és meg kell valósítania egy egyetlen üzleti képesség. 

![](./images/microservices-logical.svg)
 
Bizonyos értelemben mikroszolgáltatások célú szolgáltatás architektúrák (SOA) természetes fejlődéséhez, de mikroszolgáltatások létrehozására és SOA közötti különbségek vannak. Az alábbiakban néhány olyan mikroszolgáltatási definiáló jellemzői:

- Mikroszolgáltatások architektúra esetén a szolgáltatások kicsi, független és lazán összekapcsolt.

- Minden szolgáltatás egy külön codebase, amely egy kis fejlesztési csapat fogja kezelni akkor is.

- Szolgáltatások egymástól függetlenül is telepíthető. Egy csoport frissítheti a meglévő szolgáltatás újraépítését, és a teljes alkalmazás újbóli terjesztése nélkül.

- Szolgáltatások a saját adatok vagy a külső állapot megőrzése felelősek. Ez eltér a hagyományos modelltől, ahol egy külön adatrétege kezeli az adatmegőrzés.

- Szolgáltatások jól meghatározott API-k használatával kommunikál egymással. Belső megvalósítás részletei minden szolgáltatás más szolgáltatások rejtve maradnak.

- Szolgáltatások nem kell azonos technológia vermet, könyvtárak, vagy keretrendszert.

Emellett a szolgáltatások magukat, néhány más összetevők jelennek meg egy tipikus mikroszolgáltatások architektúra:

**Felügyeleti**. A felügyeleti összetevő felelős szolgáltatások helyezését csomópontok azonosítása sikertelen, a szolgáltatások újraelosztás csomópontok, és így tovább.  

**Felderítési szolgáltatás**.  Fenntart egy listát a szolgáltatások és a csomópontok találhatók-e meg. Lehetővé teszi, hogy a szolgáltatás értékét a található a végpont egy szolgáltatás számára. 

**API-átjáró**. Az API-átjáró a belépési pont az ügyfelek. Az ügyfelek ne hívja közvetlenül szolgáltatások. Ehelyett meghívják az API-átjárón, amely továbbítja a megfelelő szolgáltatásokat a háttérben futó hívása. Az API-átjáró előfordulhat, hogy több szolgáltatásokból válaszok összesítő és az összesített választ küldi vissza. 

Az API-átjáró használatának előnyei a következők:

- Leválasztja a ügyfelek szolgáltatásokból. A szolgáltatások lehetnek rendszerverzióval ellátott vagy átkerült anélkül, hogy az összes ügyfél frissítése.

-  Szolgáltatások használhatnak üzenetkezelő protokollokkal, amelyek nincsenek rövid, például az AMQP webes.

- Az API-átjáró más több területet funkciókat, például a hitelesítés, a naplózás, az SSL-lezárást és a terheléselosztás hajthat végre.

## <a name="when-to-use-this-architecture"></a>Mikor érdemes használni, ez az architektúra

Vegye figyelembe a architektúra stílus:

- A magas kiadás sebesség igénylő nagy alkalmazások.

- Összetett alkalmazások, csak magas szinten méretezhető.

- Alkalmazások gazdag tartományok vagy sok altartományokat.

- Olyan szervezet, amely tartalmaz kis fejlesztőcsapatai számára.


## <a name="benefits"></a>Előnyök 

- **Független központi telepítések**. Szolgáltatás frissítése a teljes alkalmazás üzembe helyezésével és állíthatja vissza, illetve előregörgetéseket frissítés esetén. Hibajavításokat tartalmaz, és a szolgáltatás kiadásokban még kezelhetőbbé teszik és kevésbé.

- **Független fejlesztési**. Egyetlen fejlesztőcsoportunk létrehozása, tesztelése és a szolgáltatás telepítése. A folyamatos innováció és kiadása gyorsabb ütemben történik eredménye. 

- **Kisméretű, célzott csoportok**. Csoportok egy szolgáltatás összpontosíthat. Minden szolgáltatás teszi könnyebben érthetőek legyenek kódbázisra, és a könnyebb új csoport tagjai hatékonyságának növeléséhez kisebb körét.

- **Elkülönítési fault**. Ha a szolgáltatás leáll, akkor nem vegye ki a teljes alkalmazás. Azonban, hogy nem jelenti azt elérhetővé az ingyenes. Továbbra is szeretné kövesse a bevált gyakorlatokat rugalmasság és a kialakítási minták. Lásd: [rugalmas alkalmazások az Azure-kialakítása][resiliency-overview].

- **Vegyes technológia verem**. Csoportok ki tudja választani a technológia, amely legjobban megfelel a szolgáltatásában. 

- **A részletes skálázás**. Szolgáltatások egymástól függetlenül lehet méretezni. Egy időben virtuális gépenként szolgáltatások magasabb sűrűségét azt jelenti, hogy Virtuálisgép-erőforrások teljes mértékben ki van használva. Egy elhelyezési korlátozás használ, a szolgáltatások rendelhető hozzá egy virtuális gép-profilja (magas CPU, nagy memória, és így tovább).

## <a name="challenges"></a>Kihívásai

- **Összetettsége**. A mikroszolgáltatások alkalmazások áthelyezése több részből áll az egyenértékű egységes alkalmazásnál. Minden szolgáltatás egyszerűbb, de a teljes rendszer egészének összetettebb.

- **Fejlesztési és tesztelési**. Szolgáltatásfüggőségek fejlesztésről másik módszert igényel. Meglévő eszközök nem feltétlenül tervezték együtt szolgáltatásfüggőségek. Szolgáltatás határain túlnyúló újrabontása bonyolult lehet. Is nehéz függőségei, teszteléséhez, különösen akkor, ha az alkalmazás gyorsan fejlődnek-e.

- **A cégirányítási hiánya**. Mikroszolgáltatások létrehozására decentralizált megközelítés előnye is van, de azt is problémákkal is járhat. A sok különböző nyelv és keretrendszer, az alkalmazás váló nehéz fenntartani fordulhatnak elő. Bevezetni néhány projekt kiterjedő szabványok, csapat rugalmasságot túlságosan korlátozás nélkül hasznos lehet. Ez különösen több területet funkciók, például a naplózás vonatkozik.

- **Hálózati torlódás és késés**. A sok kisméretű, a részletes szolgáltatások használatát további értekezleteire kommunikációt eredményez. Is a szolgáltatás függőségek lekérdezi a lánc túl hosszú (A B kiszolgálóra, amely meghívja a C...) hívásokról, a további várakozási probléma válhat. Szüksége lesz a Tervező API-k gondosan. Kerülje túlságosan chatty API-k, gondolja át szerializálási formátumok és használ aszinkron kommunikációt mintákat keressen.

- **Adatok sértetlensége**. Az egyes mikroszolgáltatási saját adatmegőrzés felelős. Ennek eredményeképpen a adatkonzisztencia kihívást lehet. Vezessék be a végleges konzisztencia, ahol csak lehetséges.

- **Felügyeleti**. A mikroszolgáltatások sikeres egy összetett DevOps kulturális környezet szükséges. Korrelált naplózása szolgáltatásban kihívást jelenthet. Naplózási általában több szolgáltatás hívások egyfelhasználós művelet kell összefüggéseket.

- **Versioning**. Frissítések egy kell a tőle függő szolgáltatások nem törhetik meg. Több szolgáltatások frissítése sikerült egy adott időpontban, így nélkül gondos tervezés, akkor lehet, hogy problémákat visszafelé vagy kompatibilitási továbbítsa.

- **Skillset**. Mikroszolgáltatások azok helyre elosztott rendszerek. Gondosan határozza meg, hogy a csapat a képességek és a felhasználói élmény, a sikeres-e.

## <a name="best-practices"></a>Ajánlott eljárások

- A vállalati tartomány körül modellre szolgáltatások. 

- Minden decentralize. Az egyes csoportok tervezéséről és kiépítéséről szolgáltatások felelősek. Kerülje a kódban, illetve az adatok sémák megosztása. 

- Adattárolás a szolgáltatást, amely birtokolja az adatokat a saját kell lennie. Ajánlott tárhelyet használja az egyes szolgáltatást és az adatok. 

- Szolgáltatások tetszetős API-k segítségével kommunikálnak. Kerülje a megvalósítás részletei megakadályozására. API-k modell kell a tartományhoz, a szolgáltatás nem a belső megvalósítása.

- Kerülje a szolgáltatások közötti. Kapcsolási okai a megosztott adatbázis sémák és kemény kommunikációs protokollokat.

- Több területet problémákat, például hitelesítés és az SSL-lezárást, hogy az átjáró kiürítése.

- Tartsa meg az átjáróról induló tartomány Tudásbázis. Az átjáró kell kezelni, és ügyfélkérések az üzleti szabályok vagy a tartomány logika ismeretek nélkül. Ellenkező esetben az átjáró függőség válik, és hatására a szolgáltatások közötti.

- Szolgáltatások laza kapcsoló és a magas működési Kohéziós kell rendelkezniük. Függvények, amelyek valószínűleg módosítása együtt csomagolt és együtt telepíteni kell. Ha ezek a szolgáltatások külön találhatók, ezek a szolgáltatások végül szorosan összekapcsolt, mert egy szolgáltatás változása esetén, a többi szolgáltatás frissítése. Két szolgáltatások túlságosan chatty kommunikációját szoros kapcsoló és alacsony Kohéziós tünete is lehet. 

- Különítse el a hibákat. Rugalmasság stratégiák segítségével megakadályozhatja, hogy a hibákat a szolgáltatásokon belüli egymásra épülő. Lásd: [rugalmassági minták] [ resiliency-patterns] és [rugalmas alkalmazásokhoz tervezéséhez][resiliency-overview].

## <a name="microservices-using-azure-container-service"></a>Azure Tárolószolgáltatás használatával Mikroszolgáltatások 

Azure Tárolószolgáltatás segítségével konfigurálhatja, és helyezze üzembe egy Docker-fürtöt. Az Azure tárolószolgáltatások számos népszerű tároló orchestrators, beleértve a Kubernetes, a DC/OS és a Docker Swarm támogatja.

![](./images/microservices-acs.png)
 
**Nyilvános csomópontok**. Ezek a csomópontok egy nyilvánosan elérhető terheléselosztó keresztül érhetők el. Ezek a csomópontok az API-átjáró üzemelteti.

**Háttér-csomópontok**. Ezek a csomópontok az API-átjárón keresztül elérő ügyfelek szolgáltatások futtatásához. Ezek a csomópontok nem közvetlenül fogadni az internetes forgalmat. A háttér-csomópontok közé tartozik a virtuális gépek, mindegyik különböző hardverprofilhoz egynél több alkalmazáskészlet. Például létrehozhatja a általános számítási feladatok, a magas CPU-munkaterhelések és a felső memóriaterület munkaterhelések külön készletek. 

**Felügyeleti virtuális gépek**. A főcsomópontok futtassa a tároló orchestrator virtuális gépeken. 

**Hálózati**. A nyilvános csomópontokat, a háttér-csomópontok és a felügyeleti virtuális gépek külön alhálózatokon belül ugyanazt a virtuális hálózatot (VNet) kerülnek. 

**Terheléselosztók**.  Egy külső terheléselosztót a nyilvános csomópontok előtt helyezkedik el. Internetes kérelmek nyilvános csomópontjának terjesztett. Egy másik terheléselosztó a felügyeleti virtuális gépek, virtuális gépek kezelési NAT-szabályok használata a secure shell (ssh) forgalom engedélyezésére előtt helyezkedik el.

A megbízhatósági és méretezhetőségi minden egyes szolgáltatás replikálja a rendszer több virtuális gépek között. Azonban mivel a szolgáltatások is viszonylag kis méretű (egy egységes alkalmazáshoz képest), több szolgáltatás vannak általában csomagolt be egy virtuális. Magasabb sűrűség lehetővé teszi, hogy az erőforrás-kihasználást. Ha egy adott szolgáltatáshoz jelentős erőforrásokat nem használja, nem kell az egész virtuális gép fordítsanak futnak, hogy a szolgáltatás.

Az alábbi ábrán látható négy különböző szolgáltatásokat (különböző alakzatok jelzi) futtató három csomópontot. Figyelje meg, hogy egyes szolgáltatások rendelkezik-e legalább két példányt. 
 
![](./images/microservices-node-density.png)

## <a name="microservices-using-azure-service-fabric"></a>Azure Service Fabric használatával Mikroszolgáltatások

Az alábbi ábrán egy mikroszolgáltatások architektúra Azure Service Fabric használatával.

![](./images/service-fabric.png)

A Service Fabric-fürt telepítve van egy vagy több Virtuálisgép-méretezési készlet. Lehetséges, hogy egynél több Virtuálisgép-méretezési a fürt ahhoz, hogy a Virtuálisgép-típusokon kombinációját kell beállítani. Egy API-átjáró a Service Fabric-fürt külső terheléselosztással ügyfél kérések fogadása előtt helyezkedik el.

A Service Fabric-futtatókörnyezet fürt kezelése, beleértve a szolgáltatás elhelyezését, a csomópontok feladatátvételét hiba és az állapotfigyelés hajt végre. A runtime telepítve van a fürtcsomópontokon magukat. Nincs olyan virtuális gépeket kezelő külön készletét.

Szolgáltatások kommunikálnak egymással a Service Fabric beépített fordított proxy használata. A Service Fabric egy tudja oldani az elnevezett szolgáltatás felderítési szolgáltatást biztosít.


<!-- links -->

[resiliency-overview]: ../../resiliency/index.md
[resiliency-patterns]: ../../patterns/category/resiliency.md



