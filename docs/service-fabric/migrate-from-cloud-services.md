---
title: Egy Azure Cloud Services az Azure Service Fabric-alkalmazás migrálása
description: Hogyan telepíthet át egy alkalmazást az Azure Cloud Services az Azure Service Fabric.
author: MikeWasson
ms.date: 04/11/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: 66f1431f45a0c9accf3a8227fa8cbb5966568372
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299446"
---
# <a name="migrate-an-azure-cloud-services-application-to-azure-service-fabric"></a>Egy Azure Cloud Services az Azure Service Fabric-alkalmazás migrálása 

[![GitHub](../_images/github.png) Mintakód][sample-code]

Ez a cikk ismerteti, hogy egy alkalmazás áttelepítés az Azure Cloud Services az Azure Service Fabric. Ez a architekturális döntések összpontosít, és ajánlott eljárásokat. 

Ehhez a projekthez és amelyet a lépések a Cloud Services-alkalmazás Surveys nevű ültették át a Service fabric. A cél volt a alkalmazás, néhány módosítással a lehető áttelepítéséhez. Egy újabb a cikkben azt optimalizálja az alkalmazást a Service Fabric által a mikroszolgáltatási architektúra bevezetésének.

A cikk elolvasása előtt lesz a Service Fabric- és mikroszolgáltatás-architektúrák alapjait általában megismeréséhez. Lásd az alábbi cikkeket:

- [Az Azure Service Fabric áttekintése][sf-overview]
- [Miért érdemes a mikroszolgáltatás-alapú megközelítést választani alkalmazások?][sf-why-microservices]


## <a name="about-the-surveys-application"></a>Tudnivalók a Surveys alkalmazás

A 2012-ben a minták és gyakorlatok csoport létrehozása nevű könyvekhez Surveys, nevű alkalmazás [több-bérlős alkalmazások fejlesztése a felhőhöz][tailspin-book]. A könyv tervez, és implementálja a Surveys alkalmazás Tailspin nevű fiktív vállalat ismerteti.

Felmérések egy több-bérlős alkalmazásban, amely lehetővé teszi az ügyfelek számára, hogy felmérések létrehozása. Után az alkalmazás regisztrál egy ügyfél, az ügyfél a szervezet tagjai létrehozása és közzététele felmérések, és az elemzési eredmények gyűjtése. Az alkalmazás egy nyilvános webhely, ahol személyek is igénybe vehet egy kérdőív tartalmaz. További információ az eredeti Tailspin forgatókönyv [Itt][tailspin-scenario].

Most Tailspin szeretne áthelyezni a Surveys alkalmazás mikroszolgáltatási architektúrára, az Azure-ban futó Service Fabric használatával. Az alkalmazás a Cloud Services-alkalmazás már telepítve van, mert a Tailspin a többfázisú megközelítés fogad el:

1.  Port a cloud services, Service fabric, miközben minimálisra csökkentik a módosításokat.
2.  A Service fabric, az alkalmazás optimalizálása a mikroszolgáltatási architektúra való váltással.

Ez a cikk bemutatja az első fázisa. Egy újabb cikk azt ismerteti, a második fázisa. A való életből vett projektben akkor valószínű, hogy mindkét szakaszok átfedésben lenne. A Service Fabric-re portol, miközben is el Újratervezés mikroszolgáltatások az alkalmazást. Később, előfordulhat, hogy finomíthatja az architektúra emellett talán értékkel való osztásának coarse-grained szolgáltatások kisebb szolgáltatásba.  

Az alkalmazás kódja megtalálható [GitHub][sample-code]. Ebben a tárházban a Cloud Services-alkalmazás és a Service Fabric-verziót is tartalmazza. 

> A felhőszolgáltatások, az eredeti alkalmazás frissített verzióját a *több-bérlős alkalmazások fejlesztése* címjegyzék-alkalmazásával.

## <a name="why-microservices"></a>Miért érdemes Mikroszolgáltatásokat?

Mikroszolgáltatások veszik górcső alá nem ez a cikk terjed, de Íme néhány a Tailspin reméli beszerezni a mikroszolgáltatási architektúra helyezi át a szolgáltatásokat:

- **Alkalmazásfrissítések**. Szolgáltatások egymástól függetlenül, telepíthető, így egy alkalmazás frissítéséhez, egy növekményes módszert is igénybe vehet.
- **Rugalmasság és a tartalék elkülönítési**. Ha egy szolgáltatás meghibásodik, más szolgáltatások tovább futnak.
- **Méretezhetőség**. A szolgáltatások egymástól függetlenül skálázhatók.
- **Rugalmasság**. Szolgáltatások köré üzleti forgatókönyvek esetén nem elegyét, ami megkönnyíti az új technológiák, keretrendszerek, vagy adattárakon szolgáltatások áttelepítése.
- **Rugalmas fejlesztés**. Egyes szolgáltatások kevesebb, mint a monolitikus alkalmazásoké kód, így a kód alap egyszerűbb való megértéséhez, OK, és tesztelése.
- **Kis méretű, célzott csapatok**. Az alkalmazást több kis szolgáltatásra bontani, mert a is egy kis célzott fejlesztőcsapata által készített minden szolgáltatáshoz.

## <a name="why-service-fabric"></a>Miért érdemes a Service Fabric?
      
A Service Fabric jó megoldás lehet a mikroszolgáltatási architektúra, azért, mert a Service Fabric, beleértve az elosztott rendszerekben szolgáltatásait a legtöbb beépített:

- **A fürt felügyeleti**. A Service Fabric automatikusan kezeli a csomópontok feladatátvételét hiba, szolgáltatásállapot-figyelést és más fürt felügyeleti funkcióit.
- **Horizontális skálázás**. Ha csomópontot ad hozzá egy Service Fabric-fürtöt, az alkalmazás automatikusan skálázza a kapacitást, módon szolgáltatásokat az új csomópontok között oszlanak meg.
- **A szolgáltatásészlelés**. A Service Fabric egy olyan felderítési szolgáltatást nyújt, amellyel feloldható egy elnevezett szolgáltatás végpontja.
- **Állapot nélküli és állapotalapú szolgáltatások**. Állapotalapú szolgáltatások használatát [a reliable collections][sf-reliable-collections], amely egy gyorsítótárban vagy üzenetsor helye is igénybe vehet, és lehet particionálni.
- **Alkalmazáséletciklus-kezelés**. Szolgáltatások egymástól függetlenül, az alkalmazás leállás nélkül frissíthetők.
- **Vezénylési szolgáltatás** Vezényel számítógépfürtökön.
- **Nagyobb sűrűsége** az erőforrás-használat optimalizálására. Egyetlen csomópont több szolgáltatást is üzemeltethet.

A Service Fabric egy elosztott felhőalkalmazások létrehozásához már bizonyítottan megbízható platformra így számos Microsoft-szolgáltatás, beleértve az Azure SQL Database, Cosmos DB, az Azure Event Hubs és mások, használatára szolgál. 

## <a name="comparing-cloud-services-with-service-fabric"></a>Cloud Services és a Service Fabric összehasonlítása

Az alábbi táblázatban összefoglaltuk a Cloud Services és a Service Fabric-alkalmazások közötti különbség. Részletesebb, lásd: [való migrálás előtt a Cloud Services és a Service Fabric közötti különbségekről további alkalmazások][sf-compare-cloud-services].

|        | Cloud Services | Service Fabric |
|--------|---------------|----------------|
| Alkalmazás összeállítása | Szerepkörök| Szolgáltatások |
| Sűrűség |Egy szerepkörpéldány virtuális gépenként | Az egyetlen csomópont több szolgáltatást |
| Csomópontok minimális száma | 2 / szerepkör | fürtönként, éles környezetekben üzemelő példányok 5 |
| Állapotkezelés | Állapot nélküli | Állapot nélküli vagy állapotalapú |
| Üzemeltetés | Azure | Felhőalapú vagy helyszíni. |
| Webes üzemeltetés | IIS** | Self-hosting |
| Üzemi modell | [Klasszikus üzemi modellben][azure-deployment-models] | [Resource Manager][azure-deployment-models]  |
| Csomagolás | Cloud service csomagfájlok (.cspkg) | Alkalmazás- és szolgáltatási csomagok |
| Alkalmazás frissítése | Virtuális IP-címek felcserélése vagy működés közbeni frissítés | A működés közbeni frissítés |
| Automatikus méretezés | [Beépített szolgáltatás][cloud-service-autoscale] | Az automatikus Virtuálisgép-méretezési csoportok horizontális felskálázás |
| Hibakeresés | Helyi emulátor | Helyi fürt |


\* Állapotalapú szolgáltatások használatát [a reliable collections] [ sf-reliable-collections] -állapotának tárolására replikák között, hogy olvasások helyi a fürt csomópontjaihoz. Írási műveletek replikálódnak a megbízhatóság csomópontok között. Állapotmentes szolgáltatások rendelkezhet külső állapotuk, adatbázis vagy más külső tárolási használatával.

** Feldolgozói szerepkörök is üzemeltethető helyi ASP.NET Web API OWIN-t használ.

## <a name="the-surveys-application-on-cloud-services"></a>A Surveys alkalmazás a Cloud Services

Az alábbi ábrán a Surveys alkalmazás futtatása a Cloud Services architektúráját mutatja be. 

![](./images/tailspin01.png)

Az alkalmazás két webes és feldolgozói szerepkör áll.

- A **Tailspin.Web** webes szerepkör futtatja az ASP.NET-webhely Tailspin ügyfelek által használt hozhat létre és kezelhet felmérések. Ügyfelek is használhatja ezt a webhelyet az alkalmazás regisztráljon, és előfizetéseiket. Végül Tailspin rendszergazdák használhatják azt bérlők listájának megtekintéséhez, és a bérlői adatok kezeléséhez. 

- A **Tailspin.Web.Survey.Public** webes szerepkör futtatja az ASP.NET-webhely, ahol személyek is igénybe vehet a felmérések, amelyek a Tailspin ügyfelek közzététele. 

- A **Tailspin.Workers.Survey** feldolgozói szerepkör háttérbeli feldolgozásra. A webes szerepkörök munkaelemek helyezzen be egy üzenetsort, és a feldolgozói szerepkör feldolgozza az elemeket. Két háttérfeladatokat vannak meghatározva: Azure SQL Database és a felmérésre adott válaszok kiszámításának statisztikája felmérés exportálása választ.

Mellett a Cloud Services a Surveys alkalmazás egyes más Azure-szolgáltatásokat használja:

- **Az Azure Storage** store felmérések, surveys válaszokat és bérlői kapcsolatos információkat.

- **Az Azure Redis Cache** egyes gyorsabb olvasási hozzáféréssel az Azure Storage, tárolt adatok gyorsítótárazásához. 

- **Az Azure Active Directory** hitelesítésére az ügyfelek és a Tailspin rendszergazdák (Azure AD).

- **Az Azure SQL Database** tárolására elemzés céljából a felmérés válaszokat. 

## <a name="moving-to-service-fabric"></a>A Service Fabric áthelyezése

Ahogy már említettük, a cél az ebben a fázisban a minimálisan szükséges módosításokat a Service Fabric volt áttelepítés. Ebből a célból létrehozott minden egyes felhőszolgáltatási szerepkör az eredeti alkalmazás megfelelő állapotmentes szolgáltatások:

![](./images/tailspin02.png)

Ez az architektúra szándékosan nagyon hasonlít az eredeti alkalmazás. A diagram elrejti a azonban néhány fontos eltérés. Ez a cikk a többi azt vizsgáljuk meg ezeket a különbségeket. 


## <a name="converting-the-cloud-service-roles-to-services"></a>A cloud service szerepkörök konvertálása szolgáltatások

Ahogy már említettük, hogy minden egyes felhőszolgáltatási szerepkör Service Fabric-szolgáltatás át. Mivel a felhőszolgáltatásokhoz tartozó szerepkörök állapot nélküli, ebben a szakaszban azt kézenfekvő választás a Service Fabric állapotmentes szolgáltatások létrehozásához. 

Az áttelepítés, hogy követte a lépéseket leírt [útmutató a webes és feldolgozói szerepkörök a Service Fabric állapotmentes szolgáltatások konvertálása][sf-migration]. 

### <a name="creating-the-web-front-end-services"></a>A webes előtér-szolgáltatás létrehozása

A Service Fabric segítségével a szolgáltatás fut a folyamat hozta létre a Service Fabric-futtatókörnyezet. Egy előtér-webkiszolgáló, az azt jelenti, hogy a szolgáltatás nem fut az IIS belül. Ehelyett a szolgáltatás egy webkiszolgáló kell futtatni. Ezt a módszert nevezik *helyi üzemeltetés*, mert a kódot, amely a folyamat fut a web server gazdagép funkcionál. 

Az a követelmény integrációsmodul azt jelenti, Service Fabric-szolgáltatás nem használható az ASP.NET MVC- vagy ASP.NET Web Forms, mert ezek a keretrendszerek IIS-t igényelnek, és nem támogatja a helyi tároló. A helyi üzemeltetési lehetőségek a következők:

- [ASP.NET Core][aspnet-core], saját üzemeltetésű használatával a [Kestrel] [ kestrel] webkiszolgáló. 
- [ASP.NET webes API-k][aspnet-webapi], saját üzemeltetésű használatával [OWIN][owin].
- Külső keretrendszereket, mint például [Nancy](http://nancyfx.org/).

Az eredeti Surveys alkalmazás ASP.NET MVC használ. ASP.NET MVC a Service Fabric helyi nem futhat, mert azt az alábbi áttelepítési lehetőségek figyelembe venni:

- A webes szerepkörök, az ASP.NET Core, amely lehet helyi portot.
- A webhely átalakítása egy egyoldalas alkalmazás (SPA), amely meghívja a webes API-k ASP.NET Web API használatával implementált. Ez a következőkre lenne szükség a webes kezelőfelület teljes újratervezése.
- Tartani a meglévő ASP.NET MVC-kódot, és a egy Windows Server-tárolót az IIS telepítése Service Fabricre. Ez a megközelítés kellene vagy kód módosítása nélkül. 

Az első lehetőség, az ASP.NET Core, portolása engedélyezett számunkra, hogy az ASP.NET Core a legújabb funkciók előnyeinek kihasználása. Ehhez a konvertálási, hogy követte a lépéseket, ismertetett [áttelepítése az ASP.NET MVC, ASP.NET Core MVC][aspnet-migration]. 

> [!NOTE]
> A Kestrel az ASP.NET Core használatakor egy fordított proxy forgalom az internetről, biztonsági okokból kezeléséhez a Kestrel elé helyezze. További információkért lásd: [Kestrel web server implementáció, az ASP.NET Core][kestrel]. A szakasz [az alkalmazás üzembe helyezése](#deploying-the-application) ismerteti az Azure az üzembe helyezés javasolt.

### <a name="http-listeners"></a>HTTP-figyelők

A Cloud Services, egy webes vagy feldolgozói szerepkör tesz közzé egy HTTP-végpontot a deklarálásával a [szolgáltatásdefiníciós fájl][cloud-service-endpoints]. Webes szerepkör legalább egy végponttal rendelkeznie kell.

```xml
<!-- Cloud service endpoint -->
<Endpoints>
    <InputEndpoint name="HttpIn" protocol="http" port="80" />
</Endpoints>
```

Ehhez hasonlóan a Service Fabric-végpontok a szolgáltatás jegyzékfájlban deklarált: 

```xml
<!-- Service Fabric endpoint -->
<Endpoints>
    <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" Port="8002" />
</Endpoints>
```

Egy felhőalapú szolgáltatás szerepkör eltérően azonban a Service Fabric-szolgáltatások is elhelyezhető ugyanazon a csomóponton belül. Ezért minden szolgáltatás egy különálló porttal kell figyelnie. Ez a cikk későbbi részében mutatjuk meg, hogyan ügyfélkérelmek a 80-as portot vagy a 443-as porton van irányítva a szolgáltatás a megfelelő porthoz.

Egy szolgáltatás, explicit módon létre kell hoznia figyelői végpontok. A hiba oka, hogy a Service Fabric a független kapcsolatos kommunikáció implementációt. További információkért lásd: [előtérrendszereket hozhat létre web service használata az ASP.NET Core alkalmazás][sf-aspnet-core].

## <a name="packaging-and-configuration"></a>Formátumokat támogató csomagolási és konfiguráció

 Egy felhőalapú szolgáltatás a következő konfigurációs és a csomagmegosztás fájlokat tartalmazza:

| Fájl | Leírás |
|------|-------------|
| szolgáltatás definíciós (.csdef) | A felhőszolgáltatás konfigurálása az Azure által használt beállításokat. A szerepkörök, a végpontok, az indítási feladatok és a konfigurációs beállítások neveit határozza meg. |
| szolgáltatás konfigurációs (.cscfg) | Egy központi telepítési beállítások, beleértve a szerepkörpéldányok számát, a végpont portszámokat és a konfigurációs beállítások értékeit. 
| Felhőszolgáltatás csomagjához (.cspkg) | Az alkalmazás kódja és konfigurációkat, és a szolgáltatásdefiníciós fájlt tartalmazza.  |

Van egy .csdef fájl a teljes alkalmazás esetében. Több különböző környezetek, például a local .cscfg-fájlok, tesztelési vagy éles rendelkezhet. Ha a szolgáltatás fut, a .cscfg, de a .csdef nem frissítheti. További információkért lásd: [Mi a Cloud Service-modell, és hogyan tegye Becsomagolhatja azt?][cloud-service-config]

A Service Fabric rendelkezik egy hasonló osztás szolgáltatás közötti *definíció* és a szolgáltatás *beállítások*, de a struktúra még finomabban szabályozható. Tudni, hogy a Service Fabric konfigurációs modell, ez segít megérteni, hogyan a Service Fabric-alkalmazás van csomagolva. A struktúra a következő:

```
Application package
  - Service packages
    - Code package
    - Configuration package
    - Data package (optional)
```

Az alkalmazáscsomag, üzembe helyezése. Egy vagy több szolgáltatási csomagot tartalmazza. A service-csomag kódját, konfiguráció és adatok csomagot tartalmaz. A kódcsomag tartalmazza a szolgáltatások bináris fájljait, és a konfigurációs csomagot konfigurációs beállításait tartalmazza. Ez a modell lehetővé teszi, hogy frissítse az egyes szolgáltatásokat a teljes alkalmazás újbóli telepítése nélkül. Ezenkívül lehetővé teszi az újbóli üzembe helyezés a kódot, vagy a szolgáltatás újraindítása nélkül csak a konfigurációs beállítások frissítése.

Service Fabric-alkalmazás a következő konfigurációs fájlokat tartalmazza:

| Fájl | Hely | Leírás |
|------|----------|-------------|
| ApplicationManifest.xml | Alkalmazáscsomag | Meghatározza a szolgáltatások, az alkalmazás alkotó. |
| ServiceManifest.xml | Szolgáltatási csomag| Egy vagy több szolgáltatást ismerteti. |
| Settings.xml | Konfigurációs csomag feltöltése | A szolgáltatások, a service csomagban meghatározott konfigurációs beállításait tartalmazza. |

További információkért lásd: [minta Service Fabric-alkalmazásokhoz][sf-application-model].

Támogatja a különböző konfigurációs beállítások több környezethez, használja a következő módon, ismertetett [kezelése több környezethez alkalmazásparamétereket][sf-multiple-environments]:

1. A szolgáltatás a Setting.xml fájlban adja meg a beállítást.
2. Az alkalmazásjegyzék határoz meg egy felülbírálást a beállítást.
3. Környezet-specifikus beállításokat helyezzen alkalmazásparaméter-fájlokat.


## <a name="deploying-the-application"></a>Az alkalmazás üzembe helyezése

Mivel az Azure Cloud Services egy felügyelt szolgáltatás, a Service Fabric-futtatókörnyezet a. Sok környezetben, beleértve az Azure és helyszíni Service Fabric-fürtöket hozhat létre. Ebben a cikkben koncentrálunk üzembe helyezése az Azure-bA. 

Az alábbi ábrán látható az üzembe helyezés javasolt:

![](./images/tailspin-cluster.png)

A Service Fabric-fürtöt helyezünk üzembe egy [Virtuálisgép-méretezési csoportot][vm-scale-sets]. A méretezési csoportok olyan Azure Compute-üzembe helyezéséhez és az azonos virtuális gépek kezelésére használható erőforrások. 

Ahogy említettük, a Kestrel webkiszolgáló fordított biztonsági okokból szükséges. Ez a diagram bemutatja [Azure Application Gateway][application-gateway], vagyis egy Azure-szolgáltatás, amely számos 7. rétegbeli terheléselosztási lehetőséget nyújt. Fordított proxyszolgáltatásként működik, az ügyfélkapcsolatok leállítását és a kérelmek háttérvégpontokhoz való továbbítását végzi. Használhat egy másik fordított proxy megoldással, például az nginx-et.  

### <a name="layer-7-routing"></a>Útválasztás 7. réteg

Az a [eredeti Surveys alkalmazás](https://msdn.microsoft.com/library/hh534477.aspx#sec21), a 80-as porton figyel egy webes szerepkör és a webes szerepkör 443-as porton figyel. 

| Nyilvános webhely | Felmérés felügyeleti hely |
|-------------|------------------------|
| `http://tailspin.cloudapp.net` | `https://tailspin.cloudapp.net` |

Egy másik lehetőség, hogy 7. rétegbeli útválasztási. Ebben a megközelítésben különböző URL-cím elérési első irányítja a rendszer a háttérben különböző portszámokat. Például a nyilvános hely előfordulhat, hogy használja a kezdetű URL-cím elérési utak `/public/`. 

A 7. rétegbeli útválasztási lehetőségek a következők:

- Application Gateway használatára. 

- Egy hálózati virtuális készüléket (nva-t), például az nginx-et használja.

- Egyéni az átjáró az állapotmentes szolgáltatás írási.

Fontolja meg ezt a módszert, ha már rendelkezik két vagy több szolgáltatások nyilvános HTTP-végpontokat, de rendezhet egy hely egy egyetlen tartomány nevét.

> Egy megközelítést, amiben *nem* ajánlott engedélyezi-e a révén a Service Fabric-kérelmeket küldjön a külső ügyfelek [fordított proxy][sf-reverse-proxy]. Bár ez nem lehetséges, a fordított proxy a szolgáltatások közötti kommunikációs szól. Tesz elérhetővé külső ügyfeleknek megnyitással *bármely* szolgáltatás fut a fürtben, amely rendelkezik egy HTTP-végpontot.

### <a name="node-types-and-placement-constraints"></a>Csomóponttípusok és elhelyezési korlátozások

A fenti központi telepítésben lévő összes szolgáltatás az összes csomóponton fut. Azonban is csoportosíthatók. szolgáltatások, így bizonyos szolgáltatások csak az adott csomóponton futnak. Ez a módszer használatának előnyei a következők:

- Egyes szolgáltatások futtatása a virtuális gépek különböző típusairól. Például bizonyos szolgáltatások előfordulhat, hogy nagy számítási igényű vagy gpu-k igényelnek. A Service Fabric-fürt virtuális gép különböző vegyesen használhat.
- Különítse el a háttér-szolgáltatásaikhoz, biztonsági okokból az előtér-szolgáltatásokat. Az összes előtér-szolgáltatásokat a csomópontok egy készletét fog futni, és a háttérszolgáltatások ugyanabban a fürtben lévő különböző csomópontokon fog futni.
- Különböző méretezési igényeknek. Egyes szolgáltatások igényelhet, mint a más szolgáltatások további csomópontokon való futtatáshoz. Például, ha megadja az előtérbeli és háttérbeli csomópontok, egyes egymástól függetlenül skálázhatók.

Az alábbi ábrán látható, amely elválasztja az előtérbeli és háttérbeli szolgáltatások fürt:

![](././images/node-placement.png)

Ez a megközelítés megvalósításához:

1.  A fürt létrehozásakor két vagy több csomópont típust határoznak meg. 
2.  Az egyes szolgáltatások használata [elhelyezési korlátozások] [ sf-placement-constraints] rendelhet a szolgáltatás a csomópont típusa.

Az Azure-ba történő telepítésekor mindegyik csomóponttípus külön Virtuálisgép-méretezési készlet van telepítve. A Service Fabric-fürt minden csomópont típus átnyúlik. További információkért lásd: [Service Fabric-csomóponttípusok és virtuálisgép-méretezési csoportok közötti kapcsolat][sf-node-types].

> Ha egy fürtben több csomóponttípus, csomóponttípussal elsődlegesként lett megjelölve a *elsődleges* typ uzlu. A Service Fabric futásidejű szolgáltatások, például a fürtszolgáltatás felügyeleti futtassa az elsődleges csomóponttípushoz. Az éles környezetben az elsődleges csomóponttípusnak legalább 5 csomópontok kiépítésére. A csomóponttípus rendelkeznie kell legalább 2 csomópontot.

## <a name="configuring-and-managing-the-cluster"></a>Konfigurálása és a fürt kezelése

Fürtök kell biztosítani, hogy megakadályozza a jogosulatlan felhasználók csatlakozzanak a fürtön. Javasoljuk, hogy Azure AD használata az ügyfelek és a csomópont és csomópont közötti biztonsághoz X.509 tanúsítványok hitelesítéséhez. További információkért lásd: [Service Fabric-fürtök biztonsági forgatókönyveit][sf-security].

Egy nyilvános HTTPS-végpont beállítása: [erőforrások meghatározása szolgáltatásjegyzékben][sf-manifest-resources].

Az alkalmazás horizontális virtuális gépek felvétele a fürtbe. Virtuálisgép-méretezési csoportok támogatása automatikus skálázást a teljesítményszámlálók alapján automatikus skálázási szabályok használatával. További információkért lásd: [és leskálázása automatikus skálázási szabályok használatával a Service Fabric-fürt skálázása][sf-auto-scale].

A fürt futása közben meg kell gyűjteni a központi helyet minden csomópontján. További információkért lásd: [naplók gyűjtése az Azure Diagnostics használatával][sf-logs].   


## <a name="conclusion"></a>Összegzés

A Service Fabric a Surveys alkalmazás portolása volt viszonylag egyszerű. Összefoglalva, amiket a következőket:

- Konvertálja a szerepkörök állapotmentes szolgáltatások.
- ASP.NET Core webes előtérrendszerek alakítja.
- A Service Fabric-modellt a csomagolás és a konfigurációs fájlok módosult.

Emellett a központi telepítés változása: Cloud Services – egy Virtuálisgép-méretezési csoportban futó Service Fabric-fürtön.

## <a name="next-steps"></a>További lépések

Most, hogy a Surveys alkalmazás sikeresen már Tailspin szeretné a Service Fabric segédanyaghoz, például a független szolgáltatás üzembe helyezését és verziókezelését. Ismerje meg, hogyan a Tailspin szétbontása ezek a szolgáltatások kihasználásához ezeket a Service Fabric a funkciókat az egy részletesebb architektúra [újrabontása az Azure Service Fabric-alkalmazás migrálása az Azure Cloud Services][refactor-surveys]

<!-- links -->

[application-gateway]: /azure/application-gateway/
[aspnet-core]: /aspnet/core/
[aspnet-webapi]: https://www.asp.net/web-api
[aspnet-migration]: /aspnet/core/migration/mvc
[aspnet-hosting]: /aspnet/core/fundamentals/hosting
[aspnet-webapi]: https://www.asp.net/web-api
[azure-deployment-models]: /azure/azure-resource-manager/resource-manager-deployment-model
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale-portal
[cloud-service-config]: /azure/cloud-services/cloud-services-model-and-package
[cloud-service-endpoints]: /azure/cloud-services/cloud-services-enable-communication-role-instances#worker-roles-vs-web-roles
[kestrel]: /aspnet/core/fundamentals/servers/kestrel
[lb-probes]: /azure/load-balancer/load-balancer-custom-probe-overview
[owin]: https://www.asp.net/aspnet/overview/owin-and-katana
[refactor-surveys]: refactor-migrated-app.md
[sample-code]: https://github.com/mspnp/cloud-services-to-service-fabric
[sf-application-model]: /azure/service-fabric/service-fabric-application-model
[sf-aspnet-core]: /azure/service-fabric/service-fabric-add-a-web-frontend
[sf-auto-scale]: /azure/service-fabric/service-fabric-cluster-scale-up-down
[sf-compare-cloud-services]: /azure/service-fabric/service-fabric-cloud-services-migration-differences
[sf-connect-and-communicate]: /azure/service-fabric/service-fabric-connect-and-communicate-with-services
[sf-containers]: /azure/service-fabric/service-fabric-containers-overview
[sf-logs]: /azure/service-fabric/service-fabric-diagnostics-how-to-setup-wad
[sf-manifest-resources]: /azure/service-fabric/service-fabric-service-manifest-resources
[sf-migration]: /azure/service-fabric/service-fabric-cloud-services-migration-worker-role-stateless-service
[sf-multiple-environments]: /azure/service-fabric/service-fabric-manage-multiple-environment-app-configuration
[sf-node-types]: /azure/service-fabric/service-fabric-cluster-nodetypes
[sf-overview]: /azure/service-fabric/service-fabric-overview
[sf-placement-constraints]: /azure/service-fabric/service-fabric-cluster-resource-manager-cluster-description
[sf-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[sf-reliable-services]: /azure/service-fabric/service-fabric-reliable-services-introduction
[sf-reverse-proxy]: /azure/service-fabric/service-fabric-reverseproxy
[sf-security]: /azure/service-fabric/service-fabric-cluster-security
[sf-why-microservices]: /azure/service-fabric/service-fabric-overview-microservices
[tailspin-book]: https://msdn.microsoft.com/library/ff966499.aspx
[tailspin-scenario]: https://msdn.microsoft.com/library/hh534482.aspx
[unity]: https://msdn.microsoft.com/library/ff647202.aspx
[vm-scale-sets]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
