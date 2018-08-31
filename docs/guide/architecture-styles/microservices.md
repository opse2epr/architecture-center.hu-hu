---
title: Mikroszolgáltatási architektúrastílus
description: Ismerteti a mikroszolgáltatási architektúrák előnyeit, kihívásait és ajánlott eljárásait az Azure-ban
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: fb23ac3e408f3a202d925a1bf684bc30d423f218
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325443"
---
# <a name="microservices-architecture-style"></a>Mikroszolgáltatási architektúrastílus

A mikroszolgáltatási architektúra kisebb, autonóm szolgáltatások gyűjteményéből áll. Mindegyik szolgáltatás önálló, és egyetlen üzleti képességet valósít meg. A mikroszolgáltatási architektúrák Azure-ban való létrehozásáról részletes útmutatót a [Mikroszolgáltatások tervezése, létrehozása és működtetése az Azure-ban](../../microservices/index.md) című szakaszban talál.

![](./images/microservices-logical.svg)
 
Bizonyos értelemben a mikroszolgáltatások a szolgáltatásorientált architektúrák (SOA-k) természetes továbbfejlődése, de vannak különbségek a mikroszolgáltatások és az SOA-k között. Íme néhány a mikroszolgáltatások meghatározó jellemzői közül:

- A mikroszolgáltatási architektúrában a szolgáltatások kicsik, függetlenek és lazán összekapcsoltak.

- Minden szolgáltatás egy külön kódbázis, amelyet egy kis fejlesztői csapat felügyelhet.

- A szolgáltatások egymástól függetlenül is üzembe helyezhetők. Egy csapat a meglévő szolgáltatást a teljes alkalmazás újraépítése és ismételt üzembe helyezése nélkül frissítheti.

- A szolgáltatások a saját adataik vagy külső állapotuk fenntartásáért felelősek. Ez eltér a hagyományos modelltől, ahol egy külön adatréteg kezeli az adatmegőrzést.

- A szolgáltatások jól meghatározott API-k használatával kommunikálnak egymással. Az egyes szolgáltatások belső megvalósítási részletei rejtve vannak a többi szolgáltatás elől.

- Nem kell, hogy a szolgáltatások ugyanazt a technológiát, kódtárat vagy keretrendszert használják.

A szolgáltatások mellett általában néhány egyéb összetevő is megjelenik a mikroszolgáltatási architektúrákban:

**Felügyelet**. A felügyeleti szolgáltatás felelős például a szolgáltatások csomópontra helyezéséért, a hibák azonosításáért, a szolgáltatások újraegyensúlyozásáért a csomópontokon, stb.  

**Szolgáltatásészlelés**.  Listát tart fenn a szolgáltatásokról és arról, hogy azok melyik csomóponton találhatók. Engedélyezi a szolgáltatáskeresést egy szolgáltatás végpontjának megtalálása érdekében. 

**API-átjáró**. Az API-átjáró az ügyfelek belépési pontja. Az ügyfelek nem közvetlenül hívják meg a szolgáltatásokat. Ehelyett az API-átjárót hívják meg, amely továbbítja a hívást a háttérben található megfelelő szolgáltatásnak. Előfordulhat, hogy az API-átjáró több szolgáltatásból összesíti a válaszokat, és az összesített választ adja vissza. 

Az API-átjáró használatának előnyei a következők:

- Elválasztja az ügyfeleket a szolgáltatásoktól. A szolgáltatások az összes ügyfél frissítése nélkül elláthatók verziószámmal vagy újratervezhetők.

-  A szolgáltatások használhatnak nem webbarát üzenetküldési protokollokat (például AMQP).

- Az API-átjáró egyéb, az egész rendszert érintő funkciókat is végrehajthat, például hitelesítést, naplózást, SSL-lezárást és terheléselosztást.

## <a name="when-to-use-this-architecture"></a>Mikor érdemes ezt az architektúrát használni?

Fontolja meg ennek az architektúrastílusnak a használatát a következőkhöz:

- Kiadások gyors megjelenését igénylő, nagy méretű alkalmazások.

- Összetett alkalmazások, amelyeknek jól skálázhatónak kell lenniük.

- Részletes tartománnyal vagy több altartománnyal rendelkező alkalmazások.

- Kis fejlesztői csapatokból álló vállalatok.


## <a name="benefits"></a>Előnyök 

- **Független üzembe helyezések**. A szolgáltatásokat a teljes alkalmazás ismételt üzembe helyezése nélkül frissítheti, illetve visszavonhat vagy előregörgethet egy frissítést, ha valami probléma merül fel. A hibajavítások és a funkciók kiadása kezelhetőbb és kevésbé kockázatos.

- **Független fejlesztés**. Egy fejlesztői csapat építhet, tesztelhet és helyezhet üzembe szolgáltatásokat. Ennek eredménye a folyamatos innováció és a kiadás gyorsabb üteme. 

- **Kis méretű, célzott csapatok**. A csapatok egy szolgáltatásra összpontosíthatnak. Az egyes szolgáltatások kisebb hatóköre könnyebben átláthatóvá teszi a kódbázisokat, és az új csapattagok gyorsabban megismerkedhetnek a szolgáltatással.

- **Hibaelkülönítés**. Ha a szolgáltatás leáll, nem áll le az egész alkalmazás is. Ez azonban önmagában nem biztosít rugalmasságot. Továbbra is követnie kell a rugalmasságra vonatkozó ajánlott eljárásokat és tervezési mintákat. Lásd: [Rugalmas alkalmazások tervezése az Azure-hoz][resiliency-overview].

- **Vegyes technológiák**. A csapatok kiválaszthatják a szolgáltatásukhoz legmegfelelőbb technológiát. 

- **Részletes méretezés**. A szolgáltatások egymástól függetlenül skálázhatók. Ugyanakkor az egy virtuális gépen található szolgáltatások nagyobb sűrűsége azt jelenti, hogy a virtuálisgép-erőforrások teljesen ki vannak használva. Elhelyezési korlátozások használatával a szolgáltatások összekapcsolhatók egy virtuálisgép-profillal (magas CPU-használat, magas memóriahasználat stb.).

## <a name="challenges"></a>Problémák

- **Összetettség**. A mikroszolgáltatás-alkalmazások több mozgó részből állnak, mint az egyenértékű monolitikus alkalmazások. Az egyes szolgáltatások egyszerűbbek, de a rendszer egésze összetettebb.

- **Fejlesztés és tesztelés**. A szolgáltatásfüggőségekhez való fejlesztéshez más módszer szükséges. A meglévő eszközöket nem feltétlenül a szolgáltatásfüggőségekkel való együttműködésre tervezték. A szolgáltatás határain túlnyúló újrabontás bonyolult lehet. A szolgáltatásfüggőségek tesztelése is kihívást jelent, különösen akkor, ha az alkalmazás gyorsan fejlődik.

- **Irányítás hiánya**. A mikroszolgáltatások építésének decentralizált megközelítése rendelkezik előnyökkel, de problémákhoz is vezethet. Előfordulhat, hogy végül olyan sok különböző nyelv és keretrendszer lesz jelen, hogy nehézkessé válik az alkalmazás karbantartása. Hasznos lehet néhány projektszintű szabvány bevezetése, a csapat rugalmasságának túlzott korlátozása nélkül. Ez különösen az egész rendszert érintő funkciókra vonatkozik (például a naplózásra).

- **Hálózati torlódás és késleltetés**. A sok kis részletes szolgáltatás használata több szolgáltatások közötti kommunikációt eredményezhet. Emellett ha a szolgáltatásfüggőségek láncolata túl hosszú (A szolgáltatás meghívja a B-t, amely meghívja a C-t, stb.), a további késleltetés problémává válhat. Gondosan kell megterveznie az API-kat. Kerülje a túlságosan forgalmas API-kat, gondoljon a szerializálási formátumokra, és keressen helyet az aszinkron kommunikáció használatához.

- **Adatintegritás**. Az egyes mikroszolgáltatások a saját adataik megőrzéséért felelősek. Ennek eredményeként az adatkonzisztencia problémát jelenthet. Ahol lehetséges, támogassa a végső konzisztenciát.

- **Felügyelet**. Ahhoz, hogy sikeresen használhassa a mikroszolgáltatásokat, fejlett DevOps-kultúra szükséges. A szolgáltatások korrelált naplózása kihívást jelenthet. A naplózásnak általában több szolgáltatás-hívást kell korrelálnia egyetlen felhasználói művelethez.

- **Verziókezelés**. Egy szolgáltatás frissítéseinek tilos megszüntetnie a tőle függő szolgáltatásokat. Bármikor frissülhet akár több szolgáltatás is, ezért körültekintő tervezés nélkül problémák merülhetnek fel a visszamenőleges vagy a jövőbeli kompatibilitással kapcsolatban.

- **Készségek**. A mikroszolgáltatások több helyre elosztott rendszerek. Gondosan mérlegelje, hogy a csapat rendelkezik-e a sikerességhez szükséges szakértelemmel és tapasztalattal.

## <a name="best-practices"></a>Ajánlott eljárások

- Modellezze a szolgáltatásokat az üzleti tartomány köré. 

- Decentralizáljon mindent. Az egyes csapatok felelősek a szolgáltatások megtervezéséért és építéséért. Kerülje a kódok és adatsémák megosztását. 

- Az adattárolót csak az adatokat birtokló szolgáltatás érhesse el. Használja a legjobb tárolót minden szolgáltatáshoz és adattípushoz. 

- A szolgáltatások gondosan megtervezett API-kon keresztül kommunikálnak. Kerülje a megvalósítási részletek kiszivárgását. Az API-knak a tartományt kell modellezniük, nem a szolgáltatás belső megvalósítását.

- Kerülje a szolgáltatások összekapcsolását. Az összekapcsolás okai például a megosztott adatbázissémák és a merev kommunikációs protokollok.

- Szervezze ki az egész rendszert érintő funkciókat, például a hitelesítést vagy SSL-lezárást az átjáróhoz.

- A tartományra vonatkozó ismereteket tartsa az átjárón kívül. Az átjárónak az üzleti szabályok és a tartományi logika ismerete nélkül kell kezelnie és irányítania az ügyfélkérelmeket. Ellenkező esetben az átjáró függőségé válik és a szolgáltatások összekapcsolását okozhatja.

- A szolgáltatásoknak laza összekapcsolással és magas működési kohézióval kell rendelkezniük. A várhatóan együtt módosuló funkciókat egy csomagba kell rendezni, és együtt kell üzembe helyezni. Ha ezek külön szolgáltatásokban találhatók, a szolgáltatások végül szorosan össze lesznek kapcsolva, mert az egyik szolgáltatás módosítása a másik szolgáltatás frissítését igényli. A két szolgáltatás közötti, túl forgalmas kommunikáció a szoros összekapcsolás és az alacsony kohézió jele lehet. 

- Különítse el a hibákat. Használjon rugalmassági stratégiákat, hogy megelőzze a szolgáltatásban fellépő hibák halmozódását. Lásd: [Rugalmassági minták][resiliency-patterns] és [Rugalmas alkalmazások tervezése][resiliency-overview].

## <a name="microservices-using-azure-container-service"></a>Az Azure Container Service-t használó mikroszolgáltatások 

Az [Azure Container Service](/azure/container-service/) egy Docker-fürt konfigurálására és kiosztására használható. Az Azure Container Service számos népszerű tárolóvezénylőt támogat, például a Kubernetes, DC/OS és Docker Swarm tárolóvezénylőket.

![](./images/microservices-acs.png)
 
**Nyilvános csomópontok**. Ezek a csomópontok nyilvános terheléselosztón keresztül érhetők el. Az API-átjáró ezeken a csomópontokon üzemel.

**Háttércsomópontok**. Ezek a csomópontok olyan szolgáltatásokat futtatnak, amelyeket az ügyfelek az API-átjárón keresztül érnek el. Ezek a csomópontok nem fogadnak közvetlenül internetes forgalmat. A háttércsomópontok egynél több virtuálisgép-készletet tartalmazhatnak, amelyek közül mindegyik különböző hardverprofillal rendelkezik. Létrehozhat például egy külön készletet az általános számítási feladatoknak, a magas CPU-használatú számítási feladatoknak és a nagy memóriaigényű számítási feladatoknak. 

**Felügyeleti virtuális gépek**. Ezek a virtuális gépek a tárolóvezénylők főcsomópontjait futtatják. 

**Hálózatkezelés**. A nyilvános csomópontok, a háttércsomópontok és a felügyeleti virtuális gépek külön alhálózatra kerülnek, ugyanazon a virtuális hálózaton (VNeten) belül. 

**Terheléselosztók**.  A kifelé irányuló terheléselosztó a nyilvános csomópontok előtt helyezkedik el. Internetes kérelmeket oszt ki a nyilvános csomópontoknak. Egy másik terheléselosztó a felügyeleti virtuális gépek elé lesz elhelyezve, hogy engedélyezze a Secure Shell- (SSH-) forgalom felügyeleti virtuális gépekre irányítását, NAT-szabályok használatával.

A megbízhatóság és skálázhatóság érdekében minden szolgáltatást több virtuális gépen replikál a rendszer. Mivel azonban a szolgáltatások is viszonylag egyszerűek (egy monolitikus alkalmazáshoz képest), egy virtuális gépen általában több szolgáltatás van elhelyezve. A magasabb sűrűség jobb erőforrás-használatot tesz lehetővé. Ha egy adott szolgáltatás nem használ sok erőforrást, nem szükséges egy teljes virtuális gépet a szolgáltatás futtatásához rendelnie.

Az alábbi ábrán három csomópont látható, amelyek négy különböző szolgáltatást futtatnak (ezeket különböző alakzatok jelölik). Figyelje meg, hogy minden szolgáltatás legalább két példánnyal rendelkezik. 
 
![](./images/microservices-node-density.png)

## <a name="microservices-using-azure-service-fabric"></a>Az Azure Service Fabric szolgáltatást használó mikroszolgáltatások

Az alábbi ábrán az [Azure Service Fabric](/azure/service-fabric/) szolgáltatást használó mikroszolgáltatási architektúra látható.

![](./images/service-fabric.png)

A Service Fabric-fürt egy vagy több virtuálisgép-méretezési csoportra van telepítve. Előfordulhat, hogy a fürtben egynél több virtuálisgép-méretezési csoporttal rendelkezik, hogy többféle virtuálisgép-típust használhasson. Egy külső terheléselosztóval rendelkező API-átjáró helyezkedik el a Service Fabric-fürt előtt, az ügyfélkérelmek fogadásához.

A Service Fabric-futtatókörnyezet végzi a fürt kezelését, beleértve a szolgáltatások elhelyezését, a csomópont feladatátvételét és az állapotmonitorozást. A futtatókörnyezet a fürtcsomópontokon van telepítve. A fürtkezelő virtuális gépeknek nincs külön csoportja.

A szolgáltatások a Service Fabricbe épített fordított proxy használatával kommunikálnak egymással. A Service Fabric egy olyan felderítési szolgáltatást nyújt, amellyel feloldható egy elnevezett szolgáltatás végpontja.


<!-- links -->

[resiliency-overview]: ../../resiliency/index.md
[resiliency-patterns]: ../../patterns/category/resiliency.md



