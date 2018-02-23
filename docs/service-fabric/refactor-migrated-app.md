---
title: "Azure Cloud Servicesből migrált Azure Service Fabric-alkalmazások újrabontása"
description: "Egy meglévő Azure Service Fabric-alkalmazás refactor hogyan átemelt Azure Cloud Services csomag"
author: petertay
ms.date: 01/30/2018
ms.openlocfilehash: 450648fbd0b19cdc7585738701914a1ebc1ed779
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/23/2018
---
# <a name="refactor-an-azure-service-fabric-application-migrated-from-azure-cloud-services"></a>Azure Cloud Servicesből migrált Azure Service Fabric-alkalmazások újrabontása

[![GitHub](../_images/github.png) Mintakód][sample-code]

Ez a cikk ismerteti, hogy egy meglévő Azure Service Fabric-alkalmazás részletesebb architektúrát újrabontása. Ez a cikk a Tervező, csomagolása, teljesítmény és a refactored Service Fabric-alkalmazás központi telepítési szempontok összpontosít.

## <a name="scenario"></a>Forgatókönyv

Az előző cikkben leírtaknak megfelelően [áttelepítése az Azure Cloud Services alkalmazás Azure Service Fabric][migrate-from-cloud-services], a minták és gyakorlatok team könyv 2012, a folyamat a dokumentált lett létrehozva. tervezése és megvalósítása a Felhőszolgáltatások alkalmazások az Azure-ban. A címjegyzék ismerteti, amelyek szeretne létrehozni a Felhőszolgáltatások alkalmazást Dejójáték nevű fiktív vállalat **felmérések**. A felmérések alkalmazás lehetővé teszi, hogy a felhasználók létrehozása és közzététele a nyilvános válaszoló felmérések. Az alábbi ábrán látható a felmérések alkalmazás ezen verziójára architektúrájának:

![](./images/tailspin01.png)

A **Tailspin.Web** webes szerepkör egy ASP.NET MVC-helyet, amely Dejójáték felhasználói futtatja:
* Iratkozzon fel a felmérések alkalmazás
* Hozzon létre vagy egyetlen felmérés, törlése
* egyetlen kérdőívet, az eredmények megtekintése
* kérelem, hogy az SQL, felméréseket exportálva és
* megtekintheti a összesített felmérés eredményét és elemzését.

A **Tailspin.Web.Survey.Public** webes szerepkört is futtatja egy ASP.NET MVC-helyet, amely a nyilvános meglátogat felmérések kitöltéséhez. Ezeket a válaszokat a sorhoz menteni kerüljenek.

A **Tailspin.Workers.Survey** feldolgozói szerepkör hajtja végre a háttérben történő feldolgozás által fel több várólistába érkező kérelmeket.

A minták és gyakorlatok csapata hozza létre egy új projektet az alkalmazás Azure Service Fabric port. A projekt célját volt a módosítások csak a szükséges kód egy Azure Service Fabric-fürtben futó alkalmazás eléréséhez. Az eredeti webes és feldolgozói szerepkörök emiatt nem volt kiválasztott részletesebb architektúra be. Az eredményül kapott architektúra nagyon hasonlít a felhőalapú szolgáltatás alkalmazásverziónként:

![](./images/tailspin02.png)

A **Tailspin.Web** szolgáltatást az eredeti van legelterjedtebb *Tailspin.Web* webes szerepkör.

A **Tailspin.Web.Survey.Public** szolgáltatást az eredeti van legelterjedtebb *Tailspin.Web.Survey.Public* webes szerepkör.

A **Tailspin.AnswerAnalysisService** szolgáltatást az eredeti van legelterjedtebb *Tailspin.Workers.Survey* feldolgozói szerepkör.

> [!NOTE] 
> Amíg minimális kódmódosítással végzett webes és feldolgozói szerepkörök **Tailspin.Web** és **Tailspin.Web.Survey.Public** önálló gazdagép módosították egy [vércse] webes a kiszolgáló. A korábbi felmérések alkalmazás egy ASP.Net alkalmazás tárolt Interet Information Services (IIS) használatával, de nincs lehetőség, mint a Service Fabric szolgáltatás IIS futtatásához. Ezért bármilyen webkiszolgálót kell lennie arra, hogy önálló üzemeltetett, többek között [vércse]. Bizonyos esetekben a Service Fabric a tárolóban lévő IIS-t futtató lehetőség. Lásd: [tárolók használatára vonatkozó forgatókönyvek] [ container-scenarios] további információt.  

Most Dejójáték van újrabontása részletesebb architektúra felmérések alkalmazás. A újrabontása Dejójáték tartozó kifejlesztésének, hogy könnyebben fejleszthet, elkészítéséhez és a felmérések alkalmazás központi telepítése. A meglévő webes és feldolgozói szerepkörök részletesebb architektúrát decomposing, amelyet Dejójáték azt szeretné, távolítsa el a meglévő szorosan összekapcsolt kommunikáció és függőségeket a szerepkörök között.

Dejójáték más előnyöket is nyújt részletesebb architektúra felmérések alkalmazás mozgatása látja:
* Minden szolgáltatás is csomagolja független projektek kicsi kell kezelnie egy kis csoport hatóköre.
* Minden szolgáltatás egymástól függetlenül rendszerverzióval ellátott, telepített lehet.
* Minden szolgáltatás a legjobb technológiával, hogy a szolgáltatás valósítható meg. A service fabric-fürt is szerepelnek például a .net-keretrendszert, Java vagy egyéb nyelvek például C vagy C++ különböző verzióinak használatával készített szolgáltatások.
* Minden szolgáltatás egymástól függetlenül megadhat válaszoljon rá növekszik és csökken a terhelés.

> [!NOTE] 
> Több vállalat kiszolgálása nem része ennek az alkalmazásnak újrabontása. Dejójáték több vállalat kiszolgálása támogatásához több lehetőséggel rendelkezik, és lehetővé teszi a tervezési döntések később a kezdeti terv befolyásolása nélkül. Például Dejójáték fürtön belül mindegyik bérlő külön szolgáltatáspéldánynak létrehozása vagy hozzon létre egy külön fürtöt az egyes bérlők számára.

## <a name="design-considerations"></a>Kialakítási szempontok
 
Az alábbi ábra a részletesebb architektúrát átkerült felmérések alkalmazás architektúráját mutatja be:

![](./images/surveys_03.png)

**Tailspin.Web** egy önálló helyet adó Dejójáték ügyfelek felmérések létrehozásához, és felmérések eredményeinek megtekintéséhez látogasson el az ASP.NET MVC alkalmazás állapot nélküli szolgáltatás. Ez a szolgáltatás osztja meg a kódot a legtöbb a *Tailspin.Web* szolgáltatás a merültek fel a Service Fabric-alkalmazás. Amint azt korábban említettük, ez a szolgáltatás az ASP.NET core használ, és vércse használja, mint egy WebListener végrehajtási webes előtér vált.

**Tailspin.Web.Survey.Public** is egy ASP.NET MVC-helyek önálló üzemeltetéséhez állapot nélküli szolgáltatás. Felhasználók által felkeresett felmérések válasszon a listából, és töltse ki a helyet. Ez a szolgáltatás osztja meg a kódot a legtöbb a *Tailspin.Web.Survey.Public* szolgáltatás a merültek fel a Service Fabric-alkalmazás. Ez a szolgáltatás is használja az ASP.NET Core, és egy WebListener végrehajtási webes előtér vércse használatával is vált.

**Tailspin.SurveyResponseService** , hogy a tároló válaszok az Azure Blob Storage felmérés állapotalapú szolgáltatás. Azt is egyesít válaszok a felmérés analitikai adatokat. A szolgáltatás, állapotalapú szolgáltatási lett megvalósítva, mert használ egy [ReliableConcurrentQueue] [ reliable-concurrent-queue] felmérés válaszok kötegekben feldolgozni. Ez a funkció eredetileg lett megvalósítva a *Tailspin.AnswerAnalysisService* merültek fel a Service Fabric-alkalmazás szolgáltatással.

**Tailspin.SurveyManagementService** állapotmentes szolgáltatások van, hogy tárolja és beolvassa kérdőívekkel és felmérés kérdéseket. A szolgáltatás használja az Azure Blob Storage tárolóban. Ez a funkció a data access Components az eredetileg is megtörtént a *Tailspin.Web* és *Tailspin.Web.Survey.Public* merültek fel a Service Fabric-alkalmazás szolgáltatásait. Dejójáték átkerült az eredeti funkciókat ezt a szolgáltatást, hogy engedélyezi-méretezését.

**Tailspin.SurveyAnswerService** , hogy beolvassa válaszok megtekintheti, és megtekintheti a elemzési állapot nélküli szolgáltatás. A szolgáltatás is használja az Azure Blob Storage tárolóban. Ez a funkció a data access Components az eredetileg is megtörtént a *Tailspin.Web* merültek fel a Service Fabric-alkalmazás szolgáltatással. Dejójáték átkerült az eredeti funkciókat ezt a szolgáltatást, mert kevesebb terhelés vár, és kevesebb példányt erőforrásokkal való takarékoskodáshoz használni szeretné.

**Tailspin.SurveyAnalysisService** tartja fenn a felmérés válasz összegző adatokat a Redis gyorsítótár a gyors beolvasásához állapot nélküli szolgáltatás. Ez a szolgáltatás hívja a *Tailspin.SurveyResponseService* minden alkalommal, amikor egy felmérés melléket, és az új felmérés válasz oszlopneveit az az összegzett adatokat. A szolgáltatás része az eredetileg megvalósított funkciókat a *Tailspin.AnswerAnalysisService* szolgáltatás a merültek fel a Service Fabric-alkalmazás.

## <a name="stateless-versus-stateful-services"></a>Állapot nélküli és állapotalapú szolgáltatások

Az Azure Service Fabric támogatja a következő programozási modellek:
* A Vendég végrehajtható modell lehetővé teszi, hogy a szolgáltatás csomagolni, és a Service Fabric-fürt központi telepítése a végrehajtható fájlok. A Service Fabric koordinálja, és a Vendég végrehajtható végrehajtási kezeli.
* A tároló modell lehetővé teszi, hogy a tároló-lemezképekben szolgáltatások telepítését. A Service Fabric létrehozását támogatja, és a Linux kernel fölött tárolók kezelése tartalmazza, valamint a Windows Server-tárolók. 
* A megbízható szolgáltatások programozási modell lehetővé teszi, hogy állapot nélküli és állapotalapú szolgáltatások, amelyek integrálják az összes Service Fabric platform szolgáltatás létrehozásához. Állapotalapú szolgáltatások lehetővé teszik a rendszer ne tárolja őket a Service Fabric-fürt replikált állapothoz. Állapotmentes szolgáltatások azonban nem.
* A megbízható actors programozási modell lehetővé teszi, hogy a virtuális szereplő mintát megvalósító szolgáltatások létrehozásához.

Minden a szolgáltatások a felmérések alkalmazás csak megbízható állapotmentes szolgáltatásokhoz, kivéve a *Tailspin.SurveyResponseService* szolgáltatás. Ez a szolgáltatás biztosítja a [ReliableConcurrentQueue] [ reliable-concurrent-queue] felmérés válaszok akkor dolgozza fel, ha azokat. A ReliableConcurrentQueue válaszokat az Azure Blob Storage tárolási és átadott a *Tailspin.SurveyAnalysisService* elemzés céljából. Dejójáték egy ReliableConcurrentQueue úgy dönt, mert a válaszok nem igényelnek szigorú első-first out (FIFO) rendelési például az Azure Service Bus egy várólista által biztosított. Egy ReliableConcurrentQueue is célja, hogy nagyobb teljesítményt és alacsony késést várólista biztosítanak, és műveletek feldolgozásához.

Vegye figyelembe, hogy egy ReliableConcurrentQueue várólistából kivéve tételekhez megőrizni műveletek kell ideális esetben az idempotent. Ha a rendszer kivételt hoz létre egy elem feldolgozása során az üzenetsorból, azonos elem egynél többször feldolgozása történhet. Felmérések alkalmazás felmérés egyesíteni művelet választ a *Tailspin.SurveyAnalysisService* nem idempotent van, mert Dejójáték úgy döntött, hogy a felmérés analitikai adatokat csak a jelenlegi pillanatképét a analitikai adatokat, és nem kell lenniük. A felmérési válaszokat az Azure Blob Storage menteni végül megegyeznek, a felmérést végső analysis is mindig újra kell számolni megfelelően ezek az adatok a.

## <a name="communication-framework"></a>Kommunikációs keretrendszer

Minden szolgáltatás a felmérések alkalmazás RESTful webes API-k segítségével kommunikál. RESTful API-k a következő előnyöket nyújtják:
* Könnyű használat: minden service-t az ASP.Net Core MVC, amely natív módon támogatja a webes API-k létrehozásához.
* Biztonsági: Minden egyes szolgáltatás SSL nem szükséges, miközben Dejójáték igényel minden egyes szolgáltatás ehhez. 
* Versioning: ügyfelek kell írni és tesztelése egy webes API-t egy adott verziójához.

Az alkalmazás ellenőrizze felmérés szolgáltatások használata a [fordított proxy] [ reverse-proxy] Service Fabric által megvalósított. Fordított proxy olyan szolgáltatás, amely a Service Fabric-fürt mindegyik csomópontján fut, és biztosítja az végpont, automatikus újra gombra, és más csatlakozási hibák típusú kezeli. Fordított proxy használatára, minden egyes RESTful API egy adott szolgáltatáshoz kezdeményezték egy előre meghatározott fordított proxy-porton keresztül.  Például, ha a fordított proxy port van beállítva **19081**, hívása a *Tailspin.SurveyAnswerService* tehető az alábbiak szerint:

```csharp
static SurveyAnswerService()
{
    httpClient = new HttpClient
    {
        BaseAddress = new Uri("http://localhost:19081/Tailspin/SurveyAnswerService/")
    };
}
```
Fordított proxy engedélyezéséhez adjon meg egy fordított proxy portja a Service Fabric-fürt létrehozása során. További információkért lásd: [fordított proxy] [ reverse-proxy] az Azure Service Fabric.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

Dejójáték létrehozott az ASP.NET Core-szolgáltatások *Tailspin.Web* és *Tailspin.Web.Surveys.Public* Visual Studio-sablonokkal. Alapértelmezés szerint ezek-sablonjai tartalmazzák a naplózás a konzolhoz. A konzol történő naplózás tehetik a fejlesztési és hibakeresés során, de a konzol összes naplózás el kell távolítani az alkalmazás üzemi környezetben való telepítésekor.

> [!NOTE]
> Figyelés és az éles környezetben futó Service Fabric-alkalmazások diagnosztika beállításával kapcsolatos további információkért lásd: [figyelése és diagnosztikai] [ monitoring-diagnostics] az Azure Service Fabric.

Például a következő sort *startup.cs* minden webes előtér-szolgáltatások kell megjegyzésként szerepel:

```csharp
// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    //loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    //loggerFactory.AddDebug();

    app.UseMvc();
}
```

> [!NOTE]
> Ezek a sorok feltételesen kizárhatók a Visual Studio "kibocsátási" közzétételekor beállításakor.

Végezetül Dejójáték az Dejójáték alkalmazást üzemi környezetben telepíti, ha vált a Visual Studio segítségével **kiadási** mód.

## <a name="deployment-considerations"></a>Telepítési szempontok

A refactored felmérések alkalmazások állnak öt állapotmentes szolgáltatások és egy állapotalapú szolgáltatás, így a fürt tervezési annak meghatározásában, a megfelelő Virtuálisgép-méretet és a csomópontok száma korlátozott. Az a *applicationmanifest.xml* fájl, amely leírja a fürt, Dejójáték beállítása a *InstanceCount* attribútuma a *StatelessService* címkén belül, hogy az egyes -1 a szolgáltatások. A -1 érték a szolgáltatás példányának létrehozása a fürt mindegyik csomópontján Service Fabric irányítja.

> [!NOTE]
> Állapotalapú szolgáltatások megfelelő számú partíciót és az adataikat replikáit tervezésének további lépés szükséges.

Dejójáték telepíti a fürt az Azure portál használatával. A Service Fabric-fürt erőforrástípus telepíti a szükséges infrastruktúra, beleértve a Virtuálisgép-méretezési készlet és a terheléselosztó mindegyikét. Az ajánlott Virtuálisgép-méretek a Service Fabric-fürt üzembe helyezési folyamata során az Azure portálon jelennek meg. Vegye figyelembe, hogy a virtuális gépek vannak telepítve, a Virtuálisgép-méretezési készlet, mert ezek mind akár is méretezhető, és ki felhasználóként a terhelés növekedésével.

> [!NOTE]
> A fentiekben taglaltak, az áttelepített alkalmazásverziónként felmérések a két előtér-webkiszolgálók voltak önálló üzemeltetett ASP.Net Core és vércse használatával a webkiszolgáló. Amíg az áttelepített alkalmazásverziónként felmérés nem használ egy fordított proxy, erősen ajánlott például az IIS, Nginx vagy Apache egy fordított proxy használatára. További információ: [vércse web server ASP.NET core megvalósításának bemutatása][kestrel-intro].
> A refactored felmérések alkalmazásban, a két előtér-webkiszolgálók önálló üzemeltetett az ASP.Net Core segítségével [WebListener] [ weblistener] , így nem szükséges egy fordított proxy webkiszolgálót.

## <a name="next-steps"></a>További lépések

A felmérések alkalmazás kódja megtalálható [GitHub][sample-code].

Ha most elkezdi az [Azure Service Fabric][service-fabric], először állítsa be a fejlesztési környezetet, akkor letöltheti a legújabb [Azure SDK] [ azure-sdk] és a [Azure Service Fabric SDK][service-fabric-sdk]. Az SDK magában foglalja a OneBox-kezelőt, telepítheti és helyileg, a teljes F5 hibakeresés a felmérések alkalmazás teszteléséhez.

<!-- links -->
[azure-sdk]: https://azure.microsoft.com/downloads/archive-net-downloads/
[container-scenarios]: /azure/service-fabric/service-fabric-containers-overview
[kestrel]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore2x
[kestrel-intro]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore1x
[migrate-from-cloud-services]: migrate-from-cloud-services.md
[monitoring-diagnostics]: /azure/service-fabric/service-fabric-diagnostics-overview
[reliable-concurrent-queue]: /azure/service-fabric/service-fabric-reliable-services-reliable-concurrent-queue
[reverse-proxy]: /azure/service-fabric/service-fabric-reverseproxy
[sample-code]: https://github.com/mspnp/cloud-services-to-service-fabric/tree/master/servicefabric-phase-2
[service-fabric]: /azure/service-fabric/service-fabric-get-started
[service-fabric-sdk]: /azure/service-fabric/service-fabric-get-started
[weblistener]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/weblistener
