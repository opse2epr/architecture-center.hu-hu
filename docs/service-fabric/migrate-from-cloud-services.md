---
title: "Az Azure Cloud Services alkalmazás Azure Service Fabric áttelepítése"
description: "Megtudhatja, hogyan telepíthetők át az alkalmazás Azure Cloud Services Azure Service Fabric."
author: MikeWasson
ms.date: 04/27/2017
ms.openlocfilehash: ce9c138a6b093fb7f0329c619c75bd4f4aacc2e7
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/23/2018
---
# <a name="migrate-an-azure-cloud-services-application-to-azure-service-fabric"></a>Az Azure Cloud Services alkalmazás Azure Service Fabric áttelepítése 

[![GitHub](../_images/github.png) Mintakód][sample-code]

Ez a cikk ismerteti, hogy egy alkalmazás áttelepítése Azure Cloud Services Azure Service Fabric. Az architektúra döntések összpontosít, és ajánlott eljárások. 

Ebben a projektben azt egy felmérések nevű Felhőszolgáltatás alkalmazás használatába és használatát. Ez a Service Fabric. A cél volt a lehető legkevesebb módosítások az alkalmazás áttelepítéséhez. Egy újabb cikkben azt fogja az alkalmazás optimalizálása a Service Fabric egy mikroszolgáltatások architektúra elfogadásával.

A cikk elolvasása előtt kell a Service Fabric és mikroszolgáltatások architektúrák alapjait általában működésének megértése. Lásd az alábbi cikkeket:

- [Az Azure Service Fabric áttekintése][sf-overview]
- [Miért egy mikroszolgáltatások közelítik meg az alkalmazások?][sf-why-microservices]


## <a name="about-the-surveys-application"></a>A felmérések alkalmazással kapcsolatos

A 2012-ben a minták és gyakorlatok csoport létrehozása felmérésekkel, egy könyv nevű alkalmazás [több-bérlős alkalmazások fejlesztése a felhő][tailspin-book]. A címjegyzék-alkalmazásával, amely tervez, és megvalósítja az felmérések alkalmazás Dejójáték nevű fiktív vállalat ismerteti.

Felmérések egy több-bérlős alkalmazás, amely lehetővé teszi az ügyfelek felmérések létrehozásához. Miután az ügyfél előfizet az alkalmazást, az ügyfél szervezetében tagjai létrehozását és felmérések közzétételét és az eredményeket elemzéshez gyűjtése. Az alkalmazás egy nyilvános webhely, ahol személyek is igénybe vehet egy felmérés tartalmazza. További információk az eredeti Dejójáték forgatókönyv [Itt][tailspin-scenario].

Most Dejójáték szeretné a felmérések alkalmazást, a mikroszolgáltatások architektúrára épülő Azure-on futó Service Fabric használatával. Az alkalmazás már üzembe van helyezve a Felhőszolgáltatások alkalmazásként, mert a Dejójáték egy több fázisban megközelítés fogad el:

1.  Port a felhőszolgáltatások, hogy a Service Fabric, ugyanakkor minimalizálja a módosításokat.
2.  Az alkalmazás optimalizálása a Service Fabric egy mikroszolgáltatások architektúra áthelyezésével.

Ez a cikk ismerteti az első fázisban. Egy újabb cikk ismerteti, a második fázisa. Valós projektben valószínű, hogy mindkét szakaszból átfedésben lenne. A Service Fabric eljárás, miközben is kezdenie Újratervezés micro-szolgáltatások az alkalmazást. Később, előfordulhat, hogy pontosítsa az architektúra emellett esetleg felosztásával coarse-grained szolgáltatások kisebb-szolgáltatás.  

Az alkalmazás kódja megtalálható [GitHub][sample-code]. A tárház tartalmaz, a Cloud Services alkalmazás és a Service Fabric-verzió. 

> A felhőalapú szolgáltatás nem az eredeti alkalmazás frissített verzióját a *több-bérlős alkalmazások fejlesztése* címjegyzék-alkalmazásával.

## <a name="why-microservices"></a>Miért mikroszolgáltatások létrehozására?

Egy részletes ismertető a mikroszolgáltatások nem ez a cikk terjed, de itt Dejójáték reméli mikroszolgáltatások-architektúra mozgatásával megszerezni előnyöket nyújtja:

- **Alkalmazásfrissítések**. Szolgáltatások egymástól függetlenül, telepíthető, így egy alkalmazás frissítése egy növekményes módszert is igénybe vehet.
- **Rugalmasság és a hibatűrés elkülönítési**. Ha nem sikerül egy szolgáltatás, más szolgáltatásokkal tovább futnak.
- **Méretezhetőség**. Szolgáltatások egymástól függetlenül lehet méretezni.
- **Rugalmasság**. Üzleti forgatókönyvek, nem egy technológia csomagokat, így könnyebben szolgáltatások áttelepítése új technológiákat, a keretrendszer vagy tárolók körül szolgáltatást tervezték.
- **Gyors fejlesztési**. Egyes szolgáltatások kevesebb mint egy egységes alkalmazás kódot, a kód alap könnyebb megismerkedett, okát, és teszteléséhez.
- **Kisméretű, célzott csoportok**. Az alkalmazás számos kis szolgáltatás bontani, mert minden egyes szolgáltatás épül fel egy kis célzott csoportnak.

## <a name="why-service-fabric"></a>Miért Service Fabric?
      
A Service Fabric remekül beválik, ha egy mikroszolgáltatások architektúrája miatt a szükséges elosztott rendszer szolgáltatások többsége a Service Fabric, beleértve a beépített:

- **Felügyeleti fürt**. A Service Fabric automatikusan kezeli a csomópontok feladatátvételét hiba, állapotfigyelést és egyéb fürt felügyeleti funkciót.
- **Horizontális skálázás**. Amikor csomópontot ad hozzá a Service Fabric-fürt, az alkalmazás automatikusan arányosan, ahogy a szolgáltatásokat az új csomópontok különböző pontjain.
- **A szolgáltatásészlelés**. A Service Fabric egy tudja oldani az elnevezett szolgáltatás felderítési szolgáltatást biztosít.
- **Állapot nélküli és állapotalapú szolgáltatással**. Állapotalapú szolgáltatások használata [megbízható gyűjtemények][sf-reliable-collections], amely egy gyorsítótár-vagy várólista végrehajthatók, és lehet particionálni.
- **Alkalmazás-életciklus kezelésének**. Szolgáltatások frissítése egymástól függetlenül és az alkalmazás leállítása nélkül.
- **Vezénylési szolgáltatás** gépet fürtön belül.
- **Magasabb sűrűség** optimalizálási hálózatierőforrás-fogyasztás. Egyetlen csomópont, rendelkezhet több szolgáltatásra.

A Service Fabric használják különböző szolgáltatások, a Microsoft Azure SQL Database, a Cosmos DB, az Azure Event Hubs és mások számára, beleértve a bevált platformmá elosztott felhőalapú alkalmazások így. 

## <a name="comparing-cloud-services-with-service-fabric"></a>A Service Fabric felhőalapú szolgáltatások összehasonlítása

A következő táblázat összefoglalja néhány fontos Cloud Services és a Service Fabric-alkalmazások közötti különbséget. Részletesebb leírását, lásd: [áttelepítése előtt Cloud Services és a Service Fabric közötti különbségekről további alkalmazások][sf-compare-cloud-services].

|        | Cloud Services | Service Fabric |
|--------|---------------|----------------|
| Alkalmazás összeállítás | Szerepkörök| Szolgáltatások |
| Sűrűség |Egy szerepkör példánya virtuális gépenként | Az egyetlen csomópont több szolgáltatás |
| Csomópontok minimális száma | 2 / szerepkör | az üzemi környezetek fürtönként 5 |
| Állapotkezelés | Állapot nélküli | Stateless vagy állapotalapú alkalmazások és szolgáltatások |
| Hosting | Azure | Felhőalapú vagy helyszíni |
| Webtároláshoz | IIS** | Self-hosting |
| Üzemi modell | [Klasszikus telepítési modell][azure-deployment-models] | [Erőforrás-kezelő][azure-deployment-models]  |
| Csomagolás | Cloud service csomagfájlok (.cspkg) | Alkalmazás- és szervizcsomagok |
| Alkalmazás frissítése | Virtuális IP-címcsere vagy a működés közbeni frissítés | Működés közbeni frissítés |
| Automatikus méretezés | [Beépített szolgáltatás][cloud-service-autoscale] | Automatikus Virtuálisgép-méretezési készlet horizontális felskálázása |
| Hibakeresés | Helyi emulátor | Helyi fürt |


\* Állapotalapú alkalmazások és szolgáltatások használatát szolgáltatások [megbízható gyűjtemények] [ sf-reliable-collections] állapot tárolására replikák között, amelyek minden olvasási helyi a fürt csomópontjaihoz. Írás a megbízhatóság csomópontok replikálódnak. Állapotmentes szolgáltatások rendelkezhet külső állapotot, egy adatbázist, vagy más külső tárolási.

** Feldolgozói szerepkörök saját maguk is tárolhatja, ASP.NET Web API OWIN használatával.

## <a name="the-surveys-application-on-cloud-services"></a>Az felmérések alkalmazást, a Cloud Services csomag

Az alábbi ábrán az a Cloud Services felmérések alkalmazás architektúráját mutatja be. 

![](./images/tailspin01.png)

Az alkalmazás két webes és feldolgozói szerepkörök áll.

- A **Tailspin.Web** webes szerepkör egy ASP.NET-webhely Dejójáték a felhasználók által használt létrehozásához és kezeléséhez felmérések futtatja. Az ügyfelek is alkalmas erre a webhelyre iratkozzon fel az alkalmazás és az előfizetések kezelése. Végezetül Dejójáték rendszergazdái azt a bérlők listáját és kezeli a bérlői adatforgalom. 

- A **Tailspin.Web.Survey.Public** webes szerepkör futtatja egy ASP.NET-webhely, ahol személyek Dejójáték ügyfelek közzététele felmérések vehet igénybe. 

- A **Tailspin.Workers.Survey** feldolgozói szerepkör háttérbeli feldolgozás. A webes szerepkörök munkaelemek helyezze a várólista-kiszolgálóra, és a feldolgozói szerepkör feldolgozza az elemeket. Két háttérfeladatok meghatározott: felmérés exportálása megválaszolja az Azure SQL Database, és kiszámításának statisztikája megtekintheti a válaszokat.

Cloud Services mellett a felmérések alkalmazás használja a néhány más Azure-szolgáltatásokkal:

- **Az Azure Storage** tároló felmérésekkel, felmérésekkel válaszok és bérlői kapcsolatos információkat.

- **Azure Redis Cache** gyorsítótárazza a gyorsabb olvasási hozzáférés az Azure Storage tárolt adatok egy részét. 

- **Az Azure Active Directory** (az Azure AD) segítségével hitelesíti az ügyfelek és Dejójáték rendszergazdák.

- **Az Azure SQL Database** elemzés felmérés válaszait tárolásához. 

## <a name="moving-to-service-fabric"></a>A Service Fabric áthelyezése

Ahogy azt korábban említettük, a cél az ebben a fázisban a minimálisan szükséges változtatásokat a Service Fabric lett áttelepíti. Ebből a célból létrehozott minden felhőalapú szolgáltatás szerepkör az eredeti alkalmazásban megfelelő állapotmentes szolgáltatásokhoz:

![](./images/tailspin02.png)

Szándékosan Ez az architektúra nagyon hasonlít az eredeti alkalmazást. A diagram azonban néhány fontos különbség elrejti. Ez a cikk a többi azt fogja megismerkedhet ezen eltérésekhez. 


## <a name="converting-the-cloud-service-roles-to-services"></a>A felhőszolgáltatás szerepköreit konvertálása szolgáltatások

Ahogy azt korábban említettük, a Microsoft minden felhő szerepkör-szolgáltatás a Service Fabric-szolgáltatás át. Mivel a felhőszolgáltatás szerepköreit állapotmentes, ebben a szakaszban azt végrehajtott állapotmentes szolgáltatások létrehozására a Service Fabric logika. 

Az áttelepítés azt leírt lépéseket követte [útmutató a webes és feldolgozói szerepkörök konvertálása a Service Fabric állapotmentes szolgáltatások][sf-migration]. 

### <a name="creating-the-web-front-end-services"></a>A webes előtér-szolgáltatás létrehozása

A Service Fabric szolgáltatás egy folyamatot, a Service Fabric-futtatókörnyezet által létrehozott belül fut. A webes előtér, amely azt jelenti, hogy a szolgáltatás nem fut az IIS belül. Ehelyett a szolgáltatás egy webkiszolgálót kell tárolni. Ezt a módszert nevezik *önálló üzemeltető*, mert a kódot, amely belül a folyamat fut gazdaként a web server. 

Önálló gazdagép követelmény azt jelenti, hogy a Service Fabric-szolgáltatás nem használható az ASP.NET MVC vagy ASP.NET Web Forms keretrendszerre, mivel ezen keretrendszerek IIS-t igényelnek, és nem támogatják az önálló üzemeltető. Az önálló üzemeltetési lehetőségek a következők:

- [Az ASP.NET Core][aspnet-core], önálló üzemeltetett használatával a [vércse] [ kestrel] webkiszolgáló. 
- [ASP.NET webes API-k][aspnet-webapi], önálló üzemeltetett használatával [OWIN][owin].
- Többek között külső keretrendszerek [Nancy](http://nancyfx.org/).

Az eredeti felmérések alkalmazást ASP.NET MVC használja. ASP.NET MVC a Service Fabric önálló nem futhat, mert azt az alább áttelepítési lehetőségek figyelembe venni:

- A webes szerepkörök az ASP.NET Core, amely lehet önálló központi portot.
- A webhely átalakítása egy egyoldalas alkalmazás (SPA), amely meghívja a webes API-k megvalósított ASP.NET Web API használatával. Ez igényelt volna az előtér-webkiszolgáló teljes újratervezése.
- Tartsa a meglévő ASP.NET MVC-kódot, és telepítheti a Windows Server tárolóban lévő IIS Service Fabric. Ez a megközelítés igényelnének kevéssé vagy egyáltalán ne kód módosítása. Azonban [tároló támogatási] [ sf-containers] a Service Fabric jelenleg még előzetes verzió.

E szempontok alapján, azt első használatát választotta, az ASP.NET Core eljárás. Ehhez a leírt lépések azt követni [áttelepítése az ASP.NET MVC az ASP.NET Core MVC][aspnet-migration]. 

> [!NOTE]
> Ha az ASP.NET Core használ vércse, helyezze elé, biztonsági okokból az internetről érkező forgalmat fog kezelni vércse fordított proxykiszolgálójaként. További információkért lásd: [vércse web server ASP.NET Core implementációjában][kestrel]. A szakasz [az alkalmazás telepítése](#deploying-the-application) ajánlott az Azure-telepítés ismerteti.

### <a name="http-listeners"></a>HTTP-figyelője

A felhőalapú szolgáltatások egy webes vagy feldolgozói szerepkör HTTP-végponttal mutatja is deklarálni kell legyen a [szolgáltatásdefiníciós fájl][cloud-service-endpoints]. A webes szerepkörnek rendelkeznie kell legalább egy végpontot.

```xml
<!-- Cloud service endpoint -->
<Endpoints>
    <InputEndpoint name="HttpIn" protocol="http" port="80" />
</Endpoints>
```

Hasonlóképpen a Service Fabric-végpontok egy szolgáltatás jegyzékfájlban deklarált: 

```xml
<!-- Service Fabric endpoint -->
<Endpoints>
    <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" Port="8002" />
</Endpoints>
```

Egy felhőalapú szolgáltatás szerepkör eltérően azonban a Service Fabric-szolgáltatások is elhelyezhető belül ugyanahhoz a csomóponthoz. Minden szolgáltatás, ezért egy különálló porttal kell figyelni. A cikk későbbi részében mutatjuk hogyan beolvasása irányított ügyfélkéréseket a 80-as port vagy a 443-as portot a szolgáltatás a megfelelő porthoz.

Egy szolgáltatás, explicit módon létre kell hoznia figyelői végpontok. A hiba oka, hogy a Service Fabric független kommunikációs verem kapcsolatban-e. További információkért lásd: [egy szolgáltatás előtér-webkiszolgáló az ASP.NET Core segítségével az alkalmazás összeállítása][sf-aspnet-core].

## <a name="packaging-and-configuration"></a>Csomagolás és konfigurálása

 Egy felhőalapú szolgáltatás a következő konfigurációs és a csomag fájlokat tartalmazza:

| Fájl | Leírás |
|------|-------------|
| Service definition (.csdef) | A felhőszolgáltatás konfigurálása az Azure-ban használt beállításokat. Határozza meg a szerepköröket, végpontok, indítási feladatok és a konfigurációs beállítások neveit. |
| Szolgáltatás konfigurációs (.cscfg) | Egy központi telepítési beállításokat, beleértve a szerepkörpéldányok számát, a végpont portszámok és a konfigurációs beállítások értékeit. 
| Szolgáltatás csomagba (.cspkg) | Tartalmazza az alkalmazás kódjában és konfigurációk és a szolgáltatásdefiníciós fájlban.  |

Van egy .csdef fájl a teljes alkalmazás esetében. Több különböző környezetekben, például helyi .cscfg-fájlok, tesztelési vagy éles lehet. Ha a szolgáltatás fut, a .cscfg, de a .csdef nem frissíthető. További információkért lásd: [Mi a felhőalapú szolgáltatás modell, és hogyan tegye csomag azt?][cloud-service-config]

A Service Fabric rendelkezik egy hasonló osztás szolgáltatás közötti *definition* és szolgáltatás *beállítások*, de a struktúra részletesebb. Szeretné megtudni, a Service Fabric konfigurációs modell, segít megérteni, hogyan lesz csomagolva a Service Fabric-alkalmazás. Ez a struktúra:

```
Application package
  - Service packages
    - Code package
    - Configuration package
    - Data package (optional)
```

Az alkalmazáscsomag, akkor telepíteni. Egy vagy több szolgáltatás csomagokat tartalmaz. A szolgáltatáscsomagot a kódot, a konfiguráció és az adatok csomagot tartalmaz. A kódcsomag a szolgáltatások bináris fájljait tartalmazza, és a konfigurációs csomag konfigurációs beállításait tartalmazza. Ez a modell lehetővé teszi a teljes alkalmazás üzembe helyezésével egyedi szolgáltatások frissítésére. Lehetővé teszi a frissítés csak a konfigurációs beállításokat, ismételt üzembe helyezéssel a kódot, vagy a szolgáltatás újraindítása nélkül.

A Service Fabric-alkalmazás a következő konfigurációs fájlokat tartalmazza:

| Fájl | Hely | Leírás |
|------|----------|-------------|
| ApplicationManifest.xml | Alkalmazáscsomag | Határozza meg a szolgáltatásokat, amelyeket az alkalmazás összeállítása. |
| ServiceManifest.xml | Szolgáltatáscsomag| Egy vagy több szolgáltatást ismerteti. |
| Settings.XML | Konfigurációs csomag | A szolgáltatások a szolgáltatás csomagban meghatározott konfigurációs beállításait tartalmazza. |

További információkért lásd: [egy alkalmazás a Service Fabric modell][sf-application-model].

A különböző konfigurációs beállítások több környezetek támogatásához a következő módszert használja, leírt [alkalmazás paramétereinek több környezet kezelése][sf-multiple-environments]:

1. A szolgáltatás Setting.xml fájlban adja meg a beállítást.
2. Az alkalmazás jegyzékében egy felülbírálást a beállítás határozza meg.
3. Környezet beállításokat alkalmazásparaméter-fájlokat kell helyezni.


## <a name="deploying-the-application"></a>Az alkalmazás telepítése

Azure Cloud Services egy olyan felügyelt szolgáltatás, mivel a Service Fabric futtatókörnyezettel. Service Fabric-fürtök hozhat létre, sok környezetben, beleértve az Azure és a helyszínen. Ez a cikk azt összpontosítani üzembe helyezése az Azure-bA. 

Az alábbi ábrán látható javasolt üzembe helyezési forgatókönyv:

![](./images/tailspin-cluster.png)

A Service Fabric-fürt központi telepítése egy [Virtuálisgép-méretezési készlet][vm-scale-sets]. Méretezési csoportok az Azure számítási erőforrás üzembe helyezését, és az azonos virtuális gépek kezelésére használható. 

Ahogy azt korábban említettük, a vércse webkiszolgáló fordított proxy biztonsági okokból szükséges. Ez az ábra mutatja [Azure Application Gateway][application-gateway], vagyis az Azure-szolgáltatások, amelyek különböző réteg 7 terheléselosztási képességeket biztosít. Fordított proxyszolgáltatásként működik, az ügyfélkapcsolatok leállítását és a kérelmek háttérvégpontokhoz való továbbítását végzi. Egy másik fordított proxy megoldással, például az nginx használhatja.  

### <a name="layer-7-routing"></a>7 útválasztási. réteg

Az a [eredeti felmérések alkalmazás](https://msdn.microsoft.com/library/hh534477.aspx#sec21), a 80-as porton figyel egy webes szerepkör, és a 443-as porton figyel a webes szerepkör. 

| Nyilvános webhely | Felmérés felügyeleti webhely |
|-------------|------------------------|
| `http://tailspin.cloudapp.net` | `https://tailspin.cloudapp.net` |

Egy másik lehetőség, hogy használja a réteg 7 útválasztást. Ezt a módszert használja, az URL-cím elérési útja eltér a háttérben futó különböző portszámokat beolvasása irányítja. Például a nyilvános webhely használhatja kezdetű URL-cím elérési utak `/public/`. 

A réteg 7 útválasztáshoz lehetőségek a következők:

- Alkalmazásátjáró használja. 

- A hálózati virtuális készülék (NVA), például az nginx használja.

- Egy egyéni átjáró írható állapotmentes szolgáltatások.

Vegye figyelembe ezt a módszert használja, ha két vagy több szolgáltatás nyilvános HTTP végpontokon rendelkezik, de szeretné, hogy egy hely egy egyetlen tartománynév jelennek meg.

> Egy közelítik meg, hogy azt *nem* ajánlott engedélyezi, hogy a Service Fabric keresztül kérelmek küldésére külső ügyfelek [fordított proxy][sf-reverse-proxy]. Bár ez lehetséges, a fordított proxy szolgáltatások közötti kommunikációs számára készült. Nyissa meg a külső ügyfeleknek közzététele *bármely* szolgáltatás fut, amely HTTP-végponttal rendelkezik a fürtben.

### <a name="node-types-and-placement-constraints"></a>Csomóponttípusok és elhelyezési korlátozás

A fent látható, a szolgáltatások futnak a csomóponton. Azonban szerint is csoportosíthatók-szolgáltatások, hogy bizonyos szolgáltatásokat csak az adott csomóponton a fürtön belül. Ezt a módszert használja az okai:

- Egyes szolgáltatások futtatnak különböző Virtuálisgép-típusokon. Például bizonyos szolgáltatásokat lehet, hogy számítási igényű vagy Feldolgozóegységekkel igényelnek. A Service Fabric-fürt Virtuálisgép-típusokon vegyesen lehet.
- Különítse el a háttér-szolgáltatásaihoz, biztonsági okokból az előtér-szolgáltatásokat. Az előtér-szolgáltatások fog futni, a csomópontok egy készletét, és a háttérszolgáltatások ugyanabban a fürtben lévő különböző csomópontokon fog futni.
- Különböző méretkövetelményekhez. Egyes szolgáltatások módosítania kell további csomópontján, mint más szolgáltatások futtatásához. Például ha megadja az előtér-csomópontok és a háttér-csomópontok, minden is méretezhető egymástól függetlenül.

Az alábbi ábrán látható, amely elválasztja az előtér- és szolgáltatások fürt:

![](././images/node-placement.png)

Ez a megközelítés megvalósításához:

1.  Ha a fürt létrehozása két vagy több csomópont típust határoznak meg. 
2.  Az egyes szolgáltatásokhoz, használjon [placement Constraints korlátozásokat] [ sf-placement-constraints] a szolgáltatás hozzárendelése csomóponttípus.

Központi telepítésekor az Azure-ba, az egyes csomóponttípusok telepítve van egy külön Virtuálisgép-méretezési készlet. A Service Fabric-fürt minden csomópont esetében is. További információkért lásd: [Service Fabric csomóponttípusok és a virtuálisgép-méretezési csoportok közötti kapcsolat][sf-node-types].

> Ha egy fürtben több csomóponttípus, egy csomóponttípus van kijelölve a *elsődleges* csomóponttípus. A Service Fabric futásidejű szolgáltatás, a fürt felügyeleti szolgáltatás, például az elsődleges csomóponttípusok futtatható. Az elsődleges csomóponttípusok éles környezetben legalább 5-csomópont kiépítéséhez. A más típusú csomópont rendelkeznie kell legalább 2 csomópontok.

## <a name="configuring-and-managing-the-cluster"></a>A fürt kezelése és konfigurálása

Fürtök védetté kell tennie, hogy jogosulatlan felhasználók csatlakozzon a fürthöz. Javasoljuk, hogy az Azure AD segítségével hitelesíti az ügyfeleket, és az-csomópontok biztonsági X.509-tanúsítványokat. További információkért lásd: [Service Fabric-fürt biztonsági forgatókönyvek][sf-security].

Egy nyilvános HTTPS-végpont konfigurálásához lásd: [adja meg az erőforrásokat. a szolgáltatás jegyzékben][sf-manifest-resources].

Az alkalmazás horizontális virtuális gépek felvétele a fürtbe. VM skálázási készletekben támogatási automatikus skálázást alapján teljesítményszámlálók automatikus méretezése szabályokkal. További információkért lásd: [bejövő vagy kimenő automatikus méretezése szabályok használatával a Service Fabric-fürt méretezése][sf-auto-scale].

A fürt futása közben, akkor kell gyűjteni a egy központi helyen található minden csomópontot. További információkért lásd: [Naplógyűjtéshez Azure Diagnostics használatával][sf-logs].   


## <a name="conclusion"></a>Összegzés

A Service Fabric felmérések alkalmazás eljárás lett meglehetősen egyszerű. Összefoglalva, hogy a következő volt:

- A szerepkörök konvertálva állapotmentes szolgáltatásokhoz.
- Az előtér-webkiszolgálók alakítja át az ASP.NET Core.
- A csomag- és konfigurációs fájljait megváltozott a Service Fabric-modellbe.

Emellett a telepítési állapotról Felhőszolgáltatások egy Virtuálisgép-méretezési csoportban futó Service Fabric-fürt.

## <a name="next-steps"></a>További lépések

Most, hogy a felmérések alkalmazás legelterjedtebb sikeresen megtörtént, a Dejójáték szeretné kihasználni a Service Fabric-szolgáltatások, például független szolgáltatástelepítés és versioning. Ismerje meg, hogyan Dejójáték kiválasztott ezek a szolgáltatások számára, hogy kihasználja a Service Fabric funkciók részletesebb architektúrát [Azonosítóterületen egy Azure Service Fabric-alkalmazás átemelt Azure Cloud Services csomag][refactor-surveys]

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
[kestrel]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel
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
