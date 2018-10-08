---
title: Azure Cloud Servicesből migrált Azure Service Fabric-alkalmazások újrabontása
description: Hogyan bontani egy meglévő Azure Service Fabric-alkalmazás migrálása az Azure Cloud Services
author: petertay
ms.date: 01/30/2018
ms.openlocfilehash: 7b5c115acdbfca0c105e2b861af9a8049b890dca
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/05/2018
ms.locfileid: "48819074"
---
# <a name="refactor-an-azure-service-fabric-application-migrated-from-azure-cloud-services"></a>Azure Cloud Servicesből migrált Azure Service Fabric-alkalmazások újrabontása

[![GitHub](../_images/github.png) Mintakód][sample-code]

Ez a cikk ismerteti, hogy egy meglévő Azure Service Fabric-alkalmazás egy részletesebb architektúra újrabontás. Ez a cikk a tervezési, Becsomagolás, teljesítmény és telepítésével kapcsolatos megfontolások a újratervezhetők Service Fabric-alkalmazás összpontosít.

## <a name="scenario"></a>Forgatókönyv

Az előző cikkben leírt módon [áttelepítése egy Azure Cloud Services-alkalmazás Azure Service fabric][migrate-from-cloud-services], a minták és gyakorlatok csapata létrehozott egy könyv a 2012-ben, a folyamat dokumentációja amelyek segítenek a Cloud Services-alkalmazás az Azure-ban. A könyv ismerteti, amely hozzon létre egy Cloud Services nevű alkalmazást szeretne Tailspin nevű fiktív vállalat **felmérések**. A Surveys alkalmazás lehetővé teszi, hogy a felhasználók létrehozása és közzététele a felmérések, amelyek nyilvános választ. Az alábbi ábrán a Surveys alkalmazás ezen verziója architektúráját mutatja be:

![](./images/tailspin01.png)

A **Tailspin.Web** webes szerepkör futtatja az ASP.NET MVC-hely Tailspin ügyfelek által használt:
* Iratkozzon fel a Surveys alkalmazás
* létrehozása vagy törlése egy egyetlen felmérés
* egyetlen felmérést, az eredmények megtekintése
* kérés, hogy az SQL-felmérés eredménye exportálva és
* megtekintheti az összesített felmérés eredményét és elemzését.

A **Tailspin.Web.Survey.Public** webes szerepkört is futtat egy ASP.NET MVC-hely, amely a nyilvános felkeresi a felmérés kitöltéséhez. Ezek a válaszok mentését sorba kerülnek.

A **Tailspin.Workers.Survey** feldolgozói szerepkör hajtja végre a háttérben történő feldolgozás a mobilfejlesztésbe több üzenetsort érkező kérelmeket.

A mintákkal és gyakorlatokkal foglalkozó csapata hozza létre az új port az alkalmazás Azure Service fabric-projekt. Ez a projekt célja, hogy csak a szükséges kód módosításait az Azure Service Fabric-fürtben futó alkalmazás volt. Ennek eredményeképpen az eredeti webes és feldolgozói szerepköröket is nem bontva részletesebb architektúra. Az eredményül kapott architektúra nagyon hasonlít a Felhőszolgáltatás az alkalmazás verziója:

![](./images/tailspin02.png)

A **Tailspin.Web** szolgáltatást az eredeti ültették át a rendszer *Tailspin.Web* webes szerepkör.

A **Tailspin.Web.Survey.Public** szolgáltatást az eredeti ültették át a rendszer *Tailspin.Web.Survey.Public* webes szerepkör.

A **Tailspin.AnswerAnalysisService** szolgáltatást az eredeti ültették át a rendszer *Tailspin.Workers.Survey* feldolgozói szerepkörben.

> [!NOTE] 
> Közben csak minimális mennyiségű kódra módosítások történtek a webes és feldolgozói szerepkörök mindegyike **Tailspin.Web** és **Tailspin.Web.Survey.Public** úgy módosították, hogy integrációsmodul- [Kestrel] webes a kiszolgáló. A korábbi Surveys alkalmazás Interet Information Services (IIS) üzemeltetett ASP.NET-alkalmazás, de már nem az IIS-t futtató Service Fabric szolgáltatásként. Ezért bármely web kiszolgáló kell lennie arra, hogy a saját üzemeltetésű, mint például [Kestrel]. Az IIS-t futtató Service fabric egy tárolóban, bizonyos esetekben lehetőség. Lásd: [tárolók használatára vonatkozó forgatókönyvek] [ container-scenarios] további információt.  

Tailspin, az újrabontás a Surveys alkalmazás részletesebb architektúra. A tailspin motiváció, az újrabontás, hogy könnyebben fejleszthet, felépíthet és telepíthet a Surveys alkalmazás. A meglévő webes és feldolgozói szerepkörök részletesebb architektúra decomposing, amelyet Tailspin szeretné távolítsa el a meglévő szorosan összekapcsolt kommunikációs és függőségeket ezek a szerepkörök között.

Tailspin számára jelenik meg más áthelyezése a Surveys alkalmazás részletesebb architektúra előnye van:
* Minden egyes szolgáltatás csomagolható be egy kis csapata által kezelt kellően kicsire hatókörrel rendelkező független projektek.
* Minden egyes szolgáltatás egymástól függetlenül rendszerverzióval ellátott, telepített lehet.
* Minden egyes is megvalósítható, hogy a szolgáltatás a legjobb technológiával. Például egy service fabric-fürt szolgáltatások különböző verzióit a .net-keretrendszert, Java vagy más C, C++ nyelv használatával is tartalmazhat.
* Minden egyes szolgáltatás egymástól függetlenül skálázhatók való reagálás a növekszik és csökken a terhelés.

> [!NOTE] 
> Több-bérlős módhoz az alkalmazás újrabontása hatókörén kívül esik. Tailspin különböző lehetőséget is kínál, támogatja a több-bérlős, és lehetővé teszi a tervezési döntéseket később a kezdeti terv befolyásolása nélkül. Tailspin például a szolgáltatások a fürtön belül minden bérlő külön példányának létrehozása vagy egy különálló fürt létrehozása az egyes bérlők számára.

## <a name="design-considerations"></a>Kialakítási szempontok
 
Az alábbi ábrán a Surveys alkalmazás részletesebb architektúra újratervezhetők architektúráját mutatja be:

![](./images/surveys_03.png)

**Tailspin.Web** önálló a Tailspin ügyfelek felmérések létrehozása és megtekintése felmérés eredményeit, látogasson el az ASP.NET MVC alkalmazást üzemeltető állapotmentes szolgáltatás. Ez a szolgáltatás a kód a legtöbb fájlmegosztások a *Tailspin.Web* szolgáltatást a merültek fel a Service Fabric-alkalmazás. Ahogy korábban említettük, a szolgáltatás használja az ASP.NET core, és a Kestrel használja, mint a WebListener megvalósítása a webes előtérrendszer értékre vált.

**Tailspin.Web.Survey.Public** is futtató önálló egy ASP.NET MVC-hely állapotmentes szolgáltatás. Felhasználók látogasson el a webhely surveys kiválasztása egy listából, és adja meg. Ez a szolgáltatás a kód a legtöbb fájlmegosztások a *Tailspin.Web.Survey.Public* szolgáltatást a merültek fel a Service Fabric-alkalmazás. A szolgáltatás is használ az ASP.NET Core, és a Kestrel használja, mint a WebListener megvalósítása a webes előtérrendszer is vált.

**Tailspin.SurveyResponseService** , hogy a tárolók válaszok az Azure Blob Storage-felmérést egy állapotalapú szolgáltatás. A felmérés elemzési adatokat nyerhet ki válaszokat is egyesíti. A szolgáltatás megvalósítása az állapotalapú szolgáltatások, mert használja egy [ReliableConcurrentQueue] [ reliable-concurrent-queue] felmérésre adott válaszok kötegekben feldolgozásához. Ez a funkció eredetileg lett megvalósítva a *Tailspin.AnswerAnalysisService* szolgáltatást a Service Fabric-alkalmazást áthelyezni.

**Tailspin.SurveyManagementService** állapotmentes szolgáltatás van, a tárolók, és lekéri felmérések és felmérési kérdéseket. A szolgáltatás használja az Azure Blob storage. Ez a funkció a data access Components az eredetileg is implementálása a *Tailspin.Web* és *Tailspin.Web.Survey.Public* merültek fel a Service Fabric-alkalmazás szolgáltatások. Tailspin refactored ezt a szolgáltatást, hogy egymástól függetlenül skálázhatja az eredeti funkciókat.

**Tailspin.SurveyAnswerService** állapotmentes szolgáltatás, hogy lekéri felmérés válaszokat, és elemzési felmérést. A szolgáltatás is használ az Azure Blob storage. Ez a funkció a data access Components az eredetileg is implementálása a *Tailspin.Web* szolgáltatást a Service Fabric-alkalmazást áthelyezni. Tailspin refactored ezt a szolgáltatást az eredeti funkciókat, mert kevesebb terhelést vár, és erőforrásokkal való takarékoskodáshoz kevesebb példányt használni szeretné.

**Tailspin.SurveyAnalysisService** állapotmentes szolgáltatás, amely továbbra is fennáll a felmérés válasz összefoglaló adatokat a Redis-gyorsítótárban gyors lekérésére. Hívja meg ezt a szolgáltatást a *Tailspin.SurveyResponseService* minden alkalommal, amikor egy kérdőív megválaszolták, és az új válasz a felmérések egyesített az összesítő adatok. Ez a szolgáltatás az eredetileg megvalósított funkciókat tartalmazza a *Tailspin.AnswerAnalysisService* szolgáltatást a merültek fel a Service Fabric-alkalmazás.

## <a name="stateless-versus-stateful-services"></a>Állapot nélküli és állapotalapú szolgáltatások

Az Azure Service Fabric támogatja a következő programozási modellekről:
* A Vendég végrehajtható modell lehetővé teszi, hogy bármilyen végrehajtható formátum, a szolgáltatás csomagolni és telepíteni kell a Service Fabric-fürtön. A Service Fabric hangolja össze, és a Vendég végrehajtható végrehajtási kezeli.
* A tároló modell lehetővé teszi a tárolórendszerképeket a szolgáltatások üzembe helyezéséhez. Service Fabric-tárolók épülő Linux-kernel tárolókat, valamint a Windows Server-tárolók létrehozása és kezelése támogatja. 
* Az a reliable services programozási modell lehetővé teszi az állapot nélküli vagy állapotalapú szolgáltatások, amelyekbe beépül az összes Service Fabric platformfunkciók létrehozását. Állapotalapú szolgáltatások lehetővé teszik a Service Fabric-fürtön tárolja a replikált állapot. Állapotmentes szolgáltatások viszont nem.
* A reliable actors programozási modell lehetővé teszi, hogy a virtuális szereplő minta megvalósítása szolgáltatások létrehozását.

A Surveys alkalmazás összes szolgáltatásban is állapot nélküli reliable services esetében, az alábbiakat kivéve a *Tailspin.SurveyResponseService* szolgáltatás. Ez a szolgáltatás valósít meg egy [ReliableConcurrentQueue] [ reliable-concurrent-queue] feldolgozni a felmérésre adott válaszok érkezésükkor. Válaszok a ReliableConcurrentQueue az Azure Blob Storage-bA mentett és átadott a *Tailspin.SurveyAnalysisService* elemzés céljából. Tailspin egy ReliableConcurrentQueue úgy dönt, mert a válaszok nem igénylik a szigorú első-first out (FIFO) rendezése például az Azure Service Bus-várólista által biztosított. Egy ReliableConcurrentQueue is célja, hogy a nagy átviteli sebességű és kis késésű várólista és eltávolítása a sorból műveleteket.

Vegye figyelembe, hogy egy ReliableConcurrentQueue el távolítva a sorból elemét megőrizni műveletnek ideális idempotensnek kellene lennie. Ha a függvény kivételt vált ki egy elem feldolgozása során az üzenetsorból, az azonos elem egynél többször is feldolgozható. A Surveys alkalmazás a felmérés egyesíteni a műveletet a adott válaszok a *Tailspin.SurveyAnalysisService* nem idempotens áll, mivel a Tailspin úgy döntött, hogy a felmérés analitikai adatokat-e az aktuális csak elemzési adatairól készült pillanatfelvételre és nem kell lenniük. A felmérés válaszok mentése az Azure Blob Storage végül konzisztens, így a felmérés végső analysis is mindig újra kell számolni megfelelően az adatokból.

## <a name="communication-framework"></a>Kommunikációs keretrendszer

Minden egyes szolgáltatás a Surveys alkalmazás RESTful webes API-k segítségével kommunikál. RESTful API-k az alábbi előnyöket kínálják:
* Könnyű használat: minden egyes szolgáltatás az ASP.NET Core MVC, amelyek natív módon támogatja a webes API-k használatával lett összeállítva.
* Biztonság: Bár mindegyikük nincs szükség az SSL, Tailspin igényel minden egyes szolgáltatás ehhez. 
* Verziókezelés: az ügyfelek is nyelven íródott, és webes API-k egy adott verzióját tesztelve lett.

Szolgáltatásokat alkalmazás ellenőrizze a felmérés felhasználása a [fordított proxy] [ reverse-proxy] Service Fabric által megvalósított. Fordított proxy egy szolgáltatása, amely a Service Fabric-fürt minden csomópontján lefut, és megadja a végpontot, az automatikus újrapróbálkozás, és kezeli a csatlakozási hibák egyéb típusú. A fordított proxy használatára, egy RESTful API-hívás egy adott szolgáltatáshoz végzett fordított proxy előre meghatározott portot használ.  Például, ha a fordított proxy portjával van beállítva **19081**, hívása a *Tailspin.SurveyAnswerService* módon lehet tenni:

```csharp
static SurveyAnswerService()
{
    httpClient = new HttpClient
    {
        BaseAddress = new Uri("http://localhost:19081/Tailspin/SurveyAnswerService/")
    };
}
```
Ahhoz, hogy a fordított proxy, adja meg a Service Fabric-fürt létrehozása során egy fordított proxy portjával. További információkért lásd: [fordított proxy] [ reverse-proxy] az Azure Service Fabricben.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

Tailspin létrehozása az ASP.NET Core-szolgáltatások *Tailspin.Web* és *Tailspin.Web.Surveys.Public* Visual Studio-sablonok használatával. Alapértelmezés szerint ezek a sablonok közé tartozik a naplózás a konzolba. Naplózás a konzolba is történhet a fejlesztés és hibakeresés során, de a konzol összes naplózás el kell távolítani az alkalmazás éles üzembe helyezés során.

> [!NOTE]
> Monitorozást és diagnosztikát az éles üzemben futó Service Fabric-alkalmazások beállításával kapcsolatos további információkért lásd: [monitorozási és diagnosztikai] [ monitoring-diagnostics] az Azure Service Fabric.

Ha például a következő sort *startup.cs* minden, a webes kezelőfelület-szolgáltatások kell megjegyzésként:

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
> Ezek a sorok feltételesen kizárhatók a Ha a Visual Studio beállítása "kiadási" közzétételekor.

Végül Tailspin a Tailspin alkalmazást az éles környezetben üzembe helyezte, amikor vált a Visual Studióban, hogy **kiadási** mód.

## <a name="deployment-considerations"></a>Telepítési szempontok

A újratervezhetők Surveys alkalmazás tevődik össze öt stateless services és a egy állapotalapú szolgáltatás, így a fürt tervezési korlátozódik, amely meghatározza, hogy a megfelelő virtuális gép mérete és a csomópontok számát. Az a *applicationmanifest.xml* fájlt, amely ismerteti a fürt, a Tailspin beállítása a *InstanceCount* attribútuma a *StatelessService* mínusz 1-nek minden címke a szolgáltatások. A -1 érték arra utasítja a Service Fabric a szolgáltatás egy példányának létrehozása a fürt minden csomópontján.

> [!NOTE]
> Állapotalapú szolgáltatások megfelelő számú partíciókat és -replikákat az adatok az további lépés szükséges.

Tailspin üzembe helyezi a fürtöt, az Azure Portal használatával. A Service Fabric-fürt erőforrástípus összes a szükséges infrastruktúrát, beleértve a Virtuálisgép-méretezési csoportok és a egy terheléselosztót helyez üzembe. A javasolt Virtuálisgép-méretek jelennek meg az Azure Portalon a Service Fabric-fürthöz az üzembe helyezési folyamat során. Vegye figyelembe, hogy a virtuális gépek vannak üzembe helyezve egy Virtuálisgép-méretezési csoportot, mert azok is is vertikálisan fel, és ki felhasználóként a terhelés növekedésével.

> [!NOTE]
> Ahogy arra már korábban, a Surveys alkalmazás áttelepített verziójában a két webes előtérrendszerek voltak saját üzemeltetésű használata az ASP.NET Core és a Kestrel webkiszolgálóként. Bár a felmérés alkalmazás áttelepített verziója nem használ egy fordított proxy, erősen ajánlott, például az IIS, az nginx-et vagy az Apache fordított proxyt használ. További információ: [Kestrel web server megvalósításához az ASP.NET core bemutatása][kestrel-intro].
> A felmérések újratervezhetők alkalmazásban a két webes előtérrendszerek önálló üzemeltetik az ASP.NET Core [WebListener] [ weblistener] és a webkiszolgáló, így a fordított proxy nem szükséges.

## <a name="next-steps"></a>További lépések

A Surveys alkalmazás kódja megtalálható [GitHub][sample-code].

Ha meg van még csak most ismerkedik [Azure Service Fabric][service-fabric], először állítsa be a fejlesztési környezetet, majd töltse le a legújabb [Azure SDK] [ azure-sdk] és a [az Azure Service Fabric SDK][service-fabric-sdk]. Az SDK tartalmazza a beépített-kezelőt, telepítheti és tesztelheti a Surveys alkalmazás helyileg, a teljes F5 hibakeresést.

<!-- links -->
[azure-sdk]: https://azure.microsoft.com/downloads/archive-net-downloads/
[container-scenarios]: /azure/service-fabric/service-fabric-containers-overview
[Kestrel]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore2x
[kestrel-intro]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore1x
[migrate-from-cloud-services]: migrate-from-cloud-services.md
[monitoring-diagnostics]: /azure/service-fabric/service-fabric-diagnostics-overview
[reliable-concurrent-queue]: /azure/service-fabric/service-fabric-reliable-services-reliable-concurrent-queue
[reverse-proxy]: /azure/service-fabric/service-fabric-reverseproxy
[sample-code]: https://github.com/mspnp/cloud-services-to-service-fabric/tree/master/servicefabric-phase-2
[service-fabric]: /azure/service-fabric/service-fabric-get-started
[service-fabric-sdk]: /azure/service-fabric/service-fabric-get-started
[weblistener]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/weblistener
