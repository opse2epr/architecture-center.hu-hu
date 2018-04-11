---
title: Útmutató a háttérben futó feladatokhoz
description: A feladatok rendszert futtató, függetlenül a felhasználói felület a háttér útmutatást.
author: dragon119
ms.date: 05/24/2017
pnp.series.title: Best Practices
ms.openlocfilehash: 10c24afee4b880cfbf8ee534f4d7f945d2b046a9
ms.sourcegitcommit: 3426a9c5ed937f097725c487cf3d073ae5e2a347
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/01/2018
---
# <a name="background-jobs"></a>Háttérfeladatok
[!INCLUDE [header](../_includes/header.md)]

Számos különböző típusú alkalmazások háttérfeladatok, függetlenül a felhasználói felület (UI) futtató igényelnek. Például kötegelt feladatok, intenzív feladatokat, és a hosszú futású folyamatokat, például a munkafolyamatok. Nincs szükség felhasználói beavatkozásra – az alkalmazás végrehajtható feladatok a háttérben is indítsa el a feladatot, és folytassa a felhasználók interaktív kérelmek feldolgozásához. Ez segít az alkalmazás felhasználói felületén, amelyek javíthatják a rendelkezésre állási és interaktív a válaszhoz szükséges idő csökkentése terhelése minimalizálása érdekében.

Például ha egy alkalmazás szükséges olyan lemezképkészlet, amellyel a felhasználók feltöltése indexképének létrehozására használnak, azt is ehhez háttérfeladatként és a miniatűr mentése tárolóra, ha befejeződött – a felhasználó kell várnia a folyamat végrehajtása nélkül. Azonos módon a felhasználó egy rendelés olyan háttér munkafolyamatot, amely a sorrendben dolgozza fel a közben a felhasználói felület lehetővé teszi, hogy a felhasználó folytassa, keresse meg a webes alkalmazás is kezdeményezhető. Ha a háttérben futó feladat befejeződött, képes tárolt rendelések adatok frissítéséhez és e-mailt küldeni a felhasználót, hogy milyen sorrendben ellenőrzi.

Ha Ön vegye figyelembe, hogy háttérfeladatként feladat végrehajtására, a következő szempontokat-e a feladat futtatásához a felhasználói felület és felhasználói beavatkozás nélkül kell várjon a feladat végrehajtása. Előfordulhat, hogy a felhasználó vagy a felhasználói felület türelmet, amíg a teljesítése igénylő feladatok nem megfelelő, mint a feladatok a háttérben.

## <a name="types-of-background-jobs"></a>Háttér-feladatok típusai
Feladatok a háttérben rendszerint tartalmazza a következő típusú feladatok közül:

* Processzorigényes feladatok, például matematikai vagy strukturális modell elemzés.
* I/O-igényes feladatok, például egy sorozatát storage-tranzakció végrehajtása vagy a fájlok indexelése.
* Kötegelt feladatok, például az ütemezett frissítések vagy ütemezett feldolgozási.
* Hosszan futó munkafolyamatok, például a megrendelés teljesítésének vagy a szolgáltatások és rendszerek kiépítése.
* Ha a feladat elkészül biztonságosabb helyre feldolgozás érzékeny-adatok feldolgozása. Például előfordulhat, hogy nem szeretné a webalkalmazáson belül bizalmas adatok feldolgozására. Ehelyett használhatja mintaként, mint [forgalomirányító](http://msdn.microsoft.com/library/dn589793.aspx) egy elkülönített háttérfolyamatként, amely védett tárolóra hozzáféréssel rendelkezik az adatok átvitele.

## <a name="triggers"></a>Eseményindítók
Feladatok a háttérben számos különböző módon kezdeményezhető. Az alábbi kategóriák valamelyikébe tartoznak:

* [**Eseményindítók eseményvezérelt**](#event-driven-triggers). A feladat elindult válaszul egy eseményt, általában egy felhasználó vagy egy lépést, a munkafolyamat által végrehajtott műveletet.
* [**Ütemezés adatvezérelt eseményindítók**](#schedule-driven-triggers). A feladat ütemezés szerint alapuló időzítő meghívták. Ez lehet egy ismétlődő ütemezés szerint vagy egy egyszeri hívás, amely egy későbbi időpontra van megadva.

### <a name="event-driven-triggers"></a>Eseményindítók eseményvezérelt
Eseményvezérelt meghívása eseményindítót használ a háttér-feladat indítása. Eseményindítók eseményvezérelt használatával például:

* A felhasználói felület vagy egy másik feladat helyezi el egy üzenetet a sorhoz. Az üzenet tartalmaz egy műveletet, amely került sor, például a felhasználó egy rendelés adatait. A háttérben futó feladat ennél a várakozási figyeli, és egy új üzenet megérkezése észleli. Kiolvassa az üzenetet, és a háttérben történő feldolgozás bemeneteként használja az adatok.
* A felhasználói felület vagy egy másik feladat menti, vagy frissíti a tárolási értéket. A háttérben futó feladat figyeli a tárolási és módosításokat észlel. Az adatokat olvas, és a háttérben történő feldolgozás bemeneteként használja.
* A felhasználói felület vagy egy másik feladat egy kérést küld egy végpontot, például egy HTTPS URI-azonosító, vagy egy API-t, amely fel van fedve egy webszolgáltatás. Továbbítja az adatokat, hogy a kérés részeként a háttérben futó feladat végrehajtásához szükséges. A végpont vagy webes szolgáltatás meghívja a háttérfeladat, amely az adatok használja bemenetként.

Tipikus feladatok eseményvezérelt meghívása alkalmas például feldolgozásához, munkafolyamatok, a távoli szolgáltatás adatokat küld, e-mailek küldése és kiépítése a több-bérlős alkalmazások új felhasználói lemezkép.

### <a name="schedule-driven-triggers"></a>Ütemezés adatvezérelt eseményindítók
Ütemezés adatvezérelt meghívása egy számlálót használja a háttér-feladat indítása. Ütemezés adatvezérelt eseményindítók használatával például:

* Egy időzítőt, amely helyben fut az alkalmazáson belül, vagy az alkalmazás operációs rendszer részeként rendszeresen háttérfeladat hív meg.
* Egy számlálót egy másik alkalmazás vagy Azure Scheduler, például egy időzítő szolgáltatásának futtató kérést küld egy API-t vagy a webes szolgáltatás rendszeres időközönként. Az API-t vagy a webes szolgáltatás meghívja a háttérben futó feladat.
* Egy külön folyamatban vagy alkalmazás elindít egy számlálót, meg kell hívni a megadott késleltetés után egyszer, vagy egy adott időpontban a háttérfeladat okozó.

Tipikus feladatok ütemezése adatvezérelt meghívása alkalmas például kötegfeldolgozási rutinok (például a felhasználók a legutóbbi viselkedésük alapján kapcsolódó termékek listák frissítése), rutin adatfeldolgozási feladatok (például az indexek frissítéséhez vagy létrehozása Halmozott eredményeket), a napi jelentésekben, adatok megőrzési törlése és adatok konzisztenciájának adatelemzés ellenőrzi.

Ha egy ütemezés adatvezérelt feladatot, amely egyetlen példányt kell futnia, figyelembe a következőket:

* Ha átméretezi a számítási példányon, amely az ütemező (például egy virtuális gép használata a Windows ütemezett feladatok) fut, akkor az ütemező futtató több példányát. Ezek tudta elindítani a feladatot több példányát.
* Feladatok futtatása nagyobb, mint az ütemező események között, ha a Feladatütemező a feladat egy másik példánya kezdheti el az előző feladat futása közben.

## <a name="returning-results"></a>Adatszolgáltató eredmények
Feladatok a háttérben aszinkron külön folyamatban, vagy akár egy külön helyét, a felhasználói felületén vagy a folyamat a háttérben futó feladat definiálásra. Ideális esetben háttérfeladatok "tűz- és elfelejti" műveleteket, és a végrehajtás folyamatban van a felhasználói felületén vagy a hívó folyamat nincs hatással. Ez azt jelenti, hogy a hívó folyamat nem vár a feladat befejezésére. Ezért az nem automatikusan észlelés, ha a feladat befejeződik.

Ha a hívó feladat folyamatban vagy a befejezési kommunikálni háttérfeladat van szüksége, ehhez meg kell valósítani egy olyan mechanizmus. Néhány példa:

* Kijelző állapotértéket tárolóra írni, amely elérhető a felhasználói felületén vagy a hívónak tevékenységhez, figyelheti vagy ellenőriznie ezt az értéket, ha szükséges. Más adatokat, amelyek a háttérfeladat a hívónak kell visszaadnia felvehetők ugyanazt a tárhelyet.
* Figyeli a felhasználói felületen vagy a hívónak válasz várólista létrehozásához. A háttérben futó feladat üzeneteket küldhetnek a várólista jelző állapot és befejezése. A háttérfeladat a hívónak kell visszaadnia adatok elhelyezhetők legyenek az üzeneteket. Ha az Azure Service Bus használ, akkor használhatja a **ReplyTo** és **CorrelationId** Tulajdonságok Ez a funkció végrehajtásához. További információkért lásd: [a Service Bus Brokered Messaging korrelációs](http://www.cloudcasts.net/devguide/Default.aspx?id=13029).
* Teszi közzé az API-t vagy a háttérben feladatból, amelyeken a felhasználói felületen vagy a hívónak állapotadatainak végpont. A válasz, amely a hívó térjen vissza a háttérfeladat adatokat tartalmazhat.
* Ezen a számon a felhasználói felületen vagy a hívónak egy API-n keresztül, előre definiált pontokon vagy a befejezési állapotának háttérfeladat rendelkezik. Ez lehet helyi keletkező vagy a közzétételi és előfizetési mechanizmus keresztül. A kérelem vagy esemény hasznos adat, amely a hívó térjen vissza a háttérfeladat tartalmazhat.

## <a name="hosting-environment"></a>Üzemeltetési környezet
Számos különböző Azure platform szolgáltatásaiból használatával tárolhatja, háttérfeladatok:

* [**Azure-webalkalmazásokban és WebJobs**](#azure-web-apps-and-webjobs). Webjobs-feladatok segítségével számos különböző típusú parancsfájlok vagy végrehajtható programok a webalkalmazás kontextusában alapuló egyéni feladatok végrehajtása.
* [**Az Azure virtuális gépek**](#azure-virtual-machines). Ha egy Windows-szolgáltatás vagy a Windows Feladatütemező használni szeretne, esetében gyakori, dedikált virtuális gépeken a háttérben feladatok futtatásához.
* [**Az Azure Batch**](#azure-batch). Kötegelt egy platformszolgáltatás, amely a ütemezi a számítási igényű munkát futtatásához a virtuális gépek felügyelt gyűjteménye. Az automatikusan át tudja méretezni számítási erőforrásokat.
* [**Azure Tárolószolgáltatás**](#azure-container-service). Az Azure Tárolószolgáltatás egy tároló üzemeltetési környezet az Azure biztosít. 
* [**Azure Felhőszolgáltatások**](#azure-cloud-services). Írhat kódot a szerepkör, amely végrehajtja a háttérfeladatként belül.

Az alábbi szakaszok ismertetik részletesebben ezeket a beállításokat, és szempontokat tartalmaz, amelyek segítségével válassza ki a megfelelő beállítást tartalmazza.

### <a name="azure-web-apps-and-webjobs"></a>Azure-webalkalmazásokban és WebJobs

Azure WebJobs használhatja az Azure Web App háttérfeladatok egyéni feladatok végrehajtásához. Webjobs-feladatok futtatása a webalkalmazás kontextusában folyamatosan működő folyamat. Webjobs-feladatok is futtathatja egy eseményindító esemény bekövetkeztekor Azure Scheduler vagy külső tényezőkkel, például a tárolási blobokat és üzenetsorokat változásokat. Feladatok indítása és az igény szerinti leállítása és leállítása. Egy folyamatosan futó webjobs-feladat sikertelen lesz, ha a rendszer automatikusan újraindítja. Próbálkozzon újra, és hiba műveletek esetében konfigurálhatók.

Amikor konfigurál egy webjobs-feladatot:

* Ha azt szeretné, hogy a feladat egy eseményvezérelt eseményindító válaszolni, konfigurálnia kell azt **folyamatos Futtatás**. A parancsfájl vagy a program a hely/wwwroot/app_data/feladatok/folyamatos mappájában tárolja.
* Ha azt szeretné, hogy a feladat ütemezés adatvezérelt eseményindító válaszolni, konfigurálnia kell azt **ütemezhető**. A parancsfájl vagy a program hely/wwwroot/app_data/feladatok/indított mappájában tárolja.
* Ha úgy dönt, a **igény szerint futtatható** kapcsolót, ha konfigurál egy feladatot, végrehajtja a ugyanazt a kódot a **ütemezhető** indítás után lehetőséget.

Azure webjobs-feladatok futtathatók a védőfal a webalkalmazás. Ez azt jelenti, hogy ezek környezeti változók eléréséhez, és adatokat, például kapcsolati karakterláncok megosztása a webalkalmazást. A feladat hozzáfér a gépet, hogy fut a feladat egyedi azonosítója. A kapcsolódási karakterlánc **AzureWebJobsStorage** biztosít hozzáférést az Azure storage-üzenetsorok, blobok és táblák az alkalmazásadatok, és az access Service Bus üzenetkezelés és a kommunikációhoz. A kapcsolódási karakterlánc **AzureWebJobsDashboard** Feladatművelet naplófájlok hozzáférést biztosít.

Azure webjobs-feladatok a következő jellemzőkkel rendelkezik:

* **Biztonsági**: webjobs-feladatok védi a webalkalmazás üzembe helyezési hitelesítő adatokat.
* **Támogatott fájltípusok**: bash rendszerhéj-parancsfájlok (.sh), a PHP-parancsfájlok (.php), a Python-parancsfájlok (.py), a JavaScript-kódot (.js) és a végrehajtható programok (.exe és parancsfájlok (.cmd), parancsfájlok (.bat), PowerShell-parancsfájlokat (.ps1) használatával webjobs-feladatok meghatározása .jar és egyebek).
* **Központi telepítési**: használatával telepíthet parancsfájlok és végrehajtható fájlok a [Azure-portálon](/azure/app-service-web/web-sites-create-web-jobs), használatával [Visual Studio](/azure/app-service-web/websites-dotnet-deploy-webjobs), segítségével a [Azure WebJobs SDK](/azure/azure-webjobs-sdk), vagy átmásolni közvetlenül a következő helyeken:
  * Az indított végrehajtási: hely/wwwroot/app_data/feladatok/indított / {feladat neve}
  * A folyamatos végrehajtás: hely/wwwroot/app_data/feladatok/folyamatos / {feladat neve}
* **Naplózás**: Console.Out INFO (megjelölt) értéke. Console.Error Hibaként kezelt. Figyelés és diagnosztikai információk az Azure portál használatával végezheti el. Naplófájlok letöltheti közvetlenül a webhelyről. Menti a következő helyeken:
  * Az indított végrehajtási: Vfs/data/feladatok/indított/feladat neve
  * A folyamatos végrehajtás: Vfs/data/feladatok/folyamatos/feladat neve
* **Konfigurációs**: a portálon, a REST API-t és a PowerShell segítségével konfigurálhatja a webjobs-feladatok. A feladat parancsfájlként legfelső szintű könyvtárába settings.job nevű konfigurációs fájl segítségével adja meg a feladat konfigurációs információkat. Példa:
  * {"stopping_wait_time": 60}
  * {"is_singleton": true}

#### <a name="considerations"></a>Megfontolandó szempontok

* Alapértelmezés szerint a webjobs-feladatok és a webes alkalmazás mérete. Azonban konfigurálhatja úgy, hogy egyetlen példány futhat feladatok a **is_singleton** konfigurációs tulajdonság **igaz**. Egypéldányos webjobs-feladatok lehetnek hasznosak, amelyeknél nem szeretné méretezheti vagy egyidejűleg több futtató tevékenységek példányok, például újraindexelés, az adatok elemzése és a hasonló feladatok.
* A feladatok a webalkalmazás teljesítményére gyakorolt hatásának minimalizálása érdekében, célszerű egy üres Azure Web App-példány a webjobs-feladatok, amelyek hosszú ideig futniuk gazdagép vagy az erőforrás egy új App Service-csomag intenzív.

### <a name="more-information"></a>További információ
* [Azure webjobs-feladatok ajánlott erőforrások](/azure/app-service-web/websites-webjobs-resources) jeleníti meg a számos hasznos erőforrásokat, letöltéseket és minták webjobs-feladatok.

### <a name="azure-virtual-machines"></a>Azure-alapú virtuális gépek
Háttérfeladatok is elegendő lehet, hogy megakadályozza, hogy a telepített Azure Web Apps vagy Felhőszolgáltatások, vagy ezeket a beállításokat nem lehet kényelmesen használható. Tipikus többek között a központi Windows-szolgáltatások, és külső segédprogramok és végrehajtható programokat. Egy másik példa lehet egy végrehajtási környezet, amely eltér attól, hogy az alkalmazást futtató írt programot. Lehet például egy olyan Windows vagy a .NET-alkalmazás végrehajtani kívánt Unix vagy Linux programot. Az Azure virtuális gép operációs rendszerek számos lehetőségek közül választhat, és a szolgáltatás vagy a végrehajtható fájl futtatásához a virtuális gépen.

Válassza ki, mikor érdemes használni a virtuális gépek meghatározásához lásd: [Azure App Service szolgáltatások, a Cloud Services és a virtuális gépek összevetése](/azure/app-service-web/choose-web-site-cloud-service-vm/). A virtuális gépek beállításokkal kapcsolatos további információkért lásd: [Azure virtuális gép és a felhőalapú szolgáltatás mérete](http://msdn.microsoft.com/library/azure/dn197896.aspx). Az operációs rendszerek és virtuális gépek számára elérhető előre elkészített képek kapcsolatos további információkért lásd: [Azure virtuális gépek piactér](https://azure.microsoft.com/gallery/virtual-machines/).

A háttérben futó feladat egy különálló virtuális gép indításához van a lehetőségek közül:

* Közvetlenül az alkalmazásból az igény szerinti a feladatot hajthat végre egy kérést küld egy végpontot, amely a feladatütemezés elérhetővé teszi. Ez megfelel a feladathoz szükséges adatokat. Ehhez a végponthoz hívja meg a feladatot.
* Beállíthatja, hogy a feladat ütemezési vagy a választott operációs rendszerben elérhető időzítő segítségével ütemezett futtatását. Például a Windows használhatja a Windows Feladatütemező parancsfájlok és a feladatok végrehajtásához. Vagy, ha a virtuális gépen telepített SQL Server, az SQL Server Agent parancsfájlok és a feladatok végrehajtásához használhatja.
* Azure Scheduler segítségével kezdeményezni a feladat, egy üzenet, amely figyeli a feladat várólistára hozzáadásával vagy egy kérést küld, amely a feladatütemezés elérhetővé teszi az API-k.

A korábbi szakaszban [eseményindítók](#triggers) hogyan háttérfeladatok is kezdeményezhető további információt.  

#### <a name="considerations"></a>Megfontolandó szempontok
Amikor szükségességéről központi telepítése a háttérben futó feladatot egy Azure virtuális gépen, vegye figyelembe a következő szempontokat:

* Üzemeltetési háttérfeladatok külön Azure virtuális gépen, ami rugalmasságot biztosít, és lehetővé teszi a kezdeményezés, végrehajtás, ütemezés és erőforrás-elosztás pontosan meghatározhatja. Azonban futásidejű költség esetén megnöveli a virtuális gépek csak a háttér-feladatok futtatásához telepítenie kell.
* Nincs a figyelheti a feladatokat az Azure-portálon, és nincs automatikus újraindítás képes a sikertelen feladatok – Bár a virtuális gép alapszintű állapotának figyelése és használatával kezelni nem létesítmény a [Azure Resource Manager parancsmagok](https://msdn.microsoft.com/library/mt125356.aspx). Azonban nem állnak rendelkezésre folyamatok és a számítási csomópontok szálak szabályozására. Általában virtuális gépek használata szükséges további annak érdekében, hogy olyan mechanizmus, amely adatokat gyűjt a feladat instrumentation, és a virtuális gép operációs rendszerről megvalósításához. Egy olyan megoldás, amely megfelelő lehet, hogy használja a [System Center felügyeleti csomag az Azure-](https://www.microsoft.com/download/details.aspx?id=50013).
* Akkor érdemes létrehozása a figyelési mintavételek menüpontban, HTTP-végpontokról keresztül elérhetővé tett tárolókra. Ezek mintavételt kódjának sikerült Állapotellenőrzések végrehajtásához, működéssel kapcsolatos adatokat és a statisztika--gyűjthet vagy hibainformációk collate és vissza egy kezelési alkalmazás. További információkért lásd: [állapotfigyelő végpont figyelési mintát](http://msdn.microsoft.com/library/dn589789.aspx).

#### <a name="more-information"></a>További információ
* [Virtuális gépek](https://azure.microsoft.com/services/virtual-machines/) az Azure-on
* [Az Azure Virtual Machines – gyakori kérdések](/azure/virtual-machines/virtual-machines-linux-classic-faq?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)

### <a name="azure-batch"></a>Azure Batch 

Érdemes lehet [Azure Batch](/azure/batch/) kell-e nagy, párhuzamos nagy teljesítményű számítástechnikai rendszerek (HPC-) alkalmazásokat és szolgáltatásokat futtathatnak több tíz, száz vagy ezer virtuális gépek között.  

A Batch szolgáltatás kiosztja a virtuális gépek, feladatokat rendelhet a virtuális gépeket, a feladatok fut, és kíséri. Kötegelt automatikusan lehet terjeszteni a virtuális gépeket, a munkaterhelés adott válaszként. Kötegelt is biztosít a feladatütemezés. Az Azure Batch a Linux és a Windows virtuális gépeket is támogatja.

#### <a name="considerations"></a>Megfontolandó szempontok 

Kötegelt jól belsőleg párhuzamos munkaterhelések működik. Azt is párhuzamos számításokat végén csökkentse lépéssel, vagy futtassa [Message Passing Interface (MPI) alkalmazások](/azure/batch/batch-mpi) párhuzamos feladatokhoz szükséges üzenet sikeres csomópontok között. 

Egy Azure feladat fut csomópontok (VM) készletét. Egy megoldás, a készlet csak akkor, amikor szükséges, és törölje a feladat befejezése után. Ez kihasználtságát a lehető legnagyobbra, mert a csomópontok nem üresjárati, de a feladat meg kell várnia, hogy csomópontokat kiosztani. Alternatív megoldásként egy időben készletet is létrehozhat. Ezt a megközelítést minimálisra csökkenti a feladat elindításához szükséges, de a csomópontokra, amelyeket elhelyezkedik rendelkező eredményez tétlen lehet idejét. További információkért lásd: [készlet és a számítási csomópont élettartama](/azure/batch/batch-api-basics#pool-and-compute-node-lifetime).

#### <a name="more-information"></a>További információ 

* [Belsőleg párhuzamos alkalmazásokat és szolgáltatásokat futtathatnak kötegelt](/azure/batch/batch-technical-overview) 
* [Kötegelt nagyméretű párhuzamos számítási megoldások fejlesztése](/azure/batch/batch-api-basics) 
* [A számítási munkaterhelés nagy méretű kötegelt és HPC megoldások](/azure/batch/batch-hpc-solutions)

### <a name="azure-container-service"></a>Azure Container Service 

Az Azure Tárolószolgáltatás lehetővé teszi, hogy konfigurálhatja, és egy fürt indexelése alkalmazások futtatásához Azure virtuális gépek kezeléséhez. Kiválaszthatja, hogy a Docker Swarm, a DC/OS vagy Kubernetes biztosít az orchestration. 

Tárolók a feladatok a háttérben futó hasznos lehet. A következő előnyöket, többek között: 

- Tárolók támogatja a nagy sűrűségű üzemeltetéséhez. Elkülönítheti a tárolóban lévő háttérfeladat során egyes virtuális gépek több tároló helyezi el.
- A tároló orchestrator terheléselosztás, a belső hálózatra és egyéb konfigurációs feladatok konfigurálása belső terheléselosztási kezeli.
- Tárolók megkezdődött, és szükség szerint leállt. 
- Azure tároló beállításjegyzék lehetővé teszi a tárolók Azure határain belül regisztrálni. Ez biztonsági, adatvédelmi és közelhatás előnyöket tartalmaz. 

#### <a name="considerations"></a>Megfontolandó szempontok

- Egy tároló orchestrator használata ismeretét igényli. Attól függően, hogy a DevOps csapat skillset ez feltétlenül nem kapcsolatos problémát.  
- Tárolószolgáltatás IaaS környezetben futtat. Azt látja, hogy a fürt egy dedikált virtuális hálózaton belüli virtuális gépek. 

#### <a name="more-information"></a>További információ 

* [Bevezetés az Azure Tárolószolgáltatás-megoldások tároló Docker-tároló](/azure/container-service/container-service-intro) 
* [Docker-tároló TITKOS nyilvántartó bemutatása](/azure/container-registry/container-registry-intro) 

### <a name="azure-cloud-services"></a>Azure Cloud Services 
A webes szerepkör vagy egy külön feldolgozói szerepkör háttérfeladatok hajthat végre. El, hogy a feldolgozói szerepkör használja, fontolja meg a méretezhetőség és a rugalmasság követelmények, a feladatütemezés élettartama-e, amikor kibocsátási ütemben történik, biztonsági, hibatűrő képesség összevonását, versengés, összetettségét, és a logikai architektúrája. További információkért lásd: [számítási erőforrás-összevonási mintát](http://msdn.microsoft.com/library/dn589778.aspx).

Többféleképpen is a Felhőszolgáltatások szerepkörön belüli háttérfeladatok megvalósításához:

* Hozzon létre egy megvalósítása a **RoleEntryPoint** osztályt a szerepkör és a módszerekkel hajtható végre a háttérben futó feladatot. A feladatok WaIISHost.exe környezetében futnak. Használhatnak a **GetSetting** metódusában a **CloudConfigurationManager** osztály betölteni a konfigurációs beállításokat. További információkért lásd: [életciklus](#lifecycle).
* Indítási feladatok segítségével hajtható végre a háttérben futó feladatot, ha az alkalmazás indítása. A feladatok a háttérben futnak kényszerítéséhez állítsa be a **taskType** tulajdonságot **háttér** (Ha ezt nem teszi meg, az alkalmazás indítási folyamat fog halt és várjon, amíg a feladat befejezéséhez). További információkért lásd: [indítási feladatok futtatása az Azure-ban](/azure/cloud-services/cloud-services-startup-tasks).
* A WebJobs SDK segítségével háttér feladatokhoz, mint az indítási feladatok indított webjobs-feladatok végrehajtása. További információkért lásd: [.NET webjobs-feladat létrehozása az Azure App Service](/azure/app-service-web/websites-dotnet-webjobs-sdk-get-started).
* Egy indítási feladat segítségével telepíthet egy Windows-szolgáltatás, amely egy vagy több háttérfeladatok végrehajtja. Meg kell adni a **taskType** tulajdonságot **háttér** , hogy a szolgáltatás hajtja végre a háttérben. További információkért lásd: [indítási feladatok futtatása az Azure-ban](/azure/cloud-services/cloud-services-startup-tasks).

A feladatok a háttérben futó a webes szerepkör fő előnye a Mentés az üzemeltetési költségek, mivel nincs központi telepítése a további szerepkörök nem kötelező.

Háttér-feladatok futtatása az a feldolgozói szerepkör több előnye is van:

* Ez lehetővé teszi, hogy az egyes szerepkör külön méretezés kezelésére. Például szükség lehet a jelenlegi terhelés támogatásához a webes szerepkör több példányát, de kevesebb példányt a feldolgozói szerepkör, amely végrehajtja a háttérben futó feladatot. Háttér tevékenység számítási példányokért elkülönítve a felhasználói felület szerepkörök skálázás, csökkentheti üzemeltetési költségek elfogadható teljesítmény megőrzése.
* A feldolgozási terhelést jelenthet a webes szerepkör alapján háttérfeladatok szervezi azt. Rugalmas maradjanak meg a webes szerepkör, amely a felhasználói Felületet biztosít, és azt jelentheti, kevesebb példányt egy adott kötet azon felhasználók kérelmeinek támogatásához szükséges.
* Lehetővé teszi aggályokat elkülönítése végrehajtásához. Minden felhasználóiszerepkör-típus is egyértelműen meghatározott és kapcsolódó feladatok meghatározott készletének megvalósítását. Így tervezéséről és a kód könnyebb karbantartása, mert kevesebb egyes kódot és az egyes szerepkörök között funkcióval.
* Bizalmas folyamatokat és az adatok elkülönítése segítséget. Például a webes szerepkörök, amelyek megvalósítják a felhasználói felület nem kell, amely felügyelt és a feldolgozói szerepkör által szabályozott adatokat. Ez akkor lehet hasznos megerősítése biztonsági, különösen akkor, ha a mintaként használhatja például a [forgalomirányító mintát](http://msdn.microsoft.com/library/dn589793.aspx).  

#### <a name="considerations"></a>Megfontolandó szempontok
A következő szempontokat vegye figyelembe, hogyan és hol kiválasztásakor háttérfeladatok telepítendő webes és feldolgozói szerepkörök Cloud Services használata esetén:

* Háttérfeladatok meglévő webes szerepkört üzemeltető egy külön feldolgozói szerepkör csak a feladatok futtatásának takaríthat meg. Azonban valószínű befolyásolhatja a teljesítményt és rendelkezésre állást, az alkalmazás, ha nagy az igény a feldolgozásra és más erőforrások. A webes szerepkör egy külön feldolgozói szerepkör használatával védi a hosszan futó vagy erőforrás-igényes háttérfeladatok hatásának.
* Ha háttérfeladatok használatával működteti a **RoleEntryPoint** osztály, egyszerűen áthelyezheti ezt egy másik szerepkört. Például ha hoz létre, hogy az osztály a webes szerepkör és annál újabb dönthet úgy, hogy szeretné-e áthelyezheti a feldolgozói szerepkörök, a feladatok futtatása a **RoleEntryPoint** osztály megvalósítása a feldolgozói szerepkör be.
* Indítási feladatok célja, hogy a program vagy parancsfájl végrehajtása. Központi telepítése a háttérben futó feladat végrehajtható programként lehet nehezebb, különösen akkor, ha a függő szerelvényeket telepítését is igényel. Központi telepítése, és adja meg a háttérben futó feladat indítási feladatok használatakor a parancsfájl segítségével könnyebben lehet.
* Kivételek háttérfeladat sikertelenségét okozó hatást különböző, attól függően, hogy a helyük felderítése módját:
  * Ha használja a **RoleEntryPoint** osztály a módszert használja, a meghiúsult feladat, akkor a szerepkört, hogy automatikusan újraindul a feladat újraindításához. Ez hatással lehet az alkalmazás rendelkezésre állását. Ennek megelőzése érdekében győződjön meg arról, hogy tartalmazza-e robusztus kivételkezelő belül a **RoleEntryPoint** osztály és a háttér-feladatokat. Indítsa újra a sikertelen lesz, amennyiben ez a megfelelő feladatokat a kód segítségével, és indítsa újra a szerepkör csak akkor, ha nem lehet szabályosan helyreállítani a hibát a kód kivételt kivételt.
  * Indítási feladatok használatakor való telepítésért felelős a feladat a végrehajtás kezelése és ellenőrzése sikertelen.
* Kezelésére és monitorozására indítási feladatok és nehezebb használnak, mint a **RoleEntryPoint** megközelítés osztályban. Az Azure WebJobs SDK azonban könnyebben kezelheti a webjobs-feladatok, amelyek indítási feladatok révén kezdeményezi az irányítópultot tartalmaz.

#### <a name="lifecycle"></a>Életciklus 
 Ha úgy dönt, hogy valósítja meg a feladatok a háttérben webes és feldolgozói szerepkörök használatával használó Felhőszolgáltatások alkalmazások esetén a **RoleEntryPoint** osztály, fontos, hogy az osztály az életciklus megismerése megfelelően használatához.

Webes és feldolgozói szerepkörök halad át fázisban olyan készlete, indítsa el, futtassa, és állítsa le. A **RoleEntryPoint** osztály közzétesz egy eseménysorozatokat, amelyek jelzik, ha a szakaszok is megjelenhetnek. Használjon ezek inicializálása, fut, és az egyéni háttér feladatok leállítása. A teljes ciklus a következő:

* Azure szerepkör szerelvény betöltése, és megkeresi az abból származó osztály **RoleEntryPoint**.
* Ha ez az osztály talál, meghívja **RoleEntryPoint.OnStart()**. Akkor bírálja felül ezt a metódust a háttérfeladatok inicializálni.
* Miután a **OnStart** metódus befejeződött, az Azure meghívja **Application_Start()** az alkalmazás globális fájl, ha ez található (például egy ASP.NET futó webes szerepkör a Global.asax).
* Az Azure hívások **RoleEntryPoint.Run()** egy új előtér-szálon, amely végrehajtja a párhuzamosan **OnStart()**. Akkor bírálja felül ezt a metódust a háttérben feladatokat indíthat el.
* A Run metódus addig tart, amíg Azure először hívja **Application_End()** a az alkalmazás globális fájlt, ha jelen, majd hívások **RoleEntryPoint.OnStop()**. Akkor bírálja felül a **OnStop** eljárás a háttérben futó feladatot karbantartása erőforrások, az objektumok eldobásakor, majd zárja be, amely a feladatok használt kapcsolatok.
* Az Azure feldolgozói szerepkör tartozó gazdafolyamat le van állítva. Ezen a ponton a szerepkör újrafelhasználásra kerül, és újra fog indulni.

A további részletek és módszerek használatának példája a **RoleEntryPoint** osztály című [számítási erőforrás-összevonási mintát](http://msdn.microsoft.com/library/dn589778.aspx).

#### <a name="implementation-considerations"></a>Megvalósítási kapcsolatos szempontok

Ha egy webes vagy feldolgozói szerepkör háttérfeladatok végrehajtási, vegye figyelembe az alábbiakat:

* Az alapértelmezett **futtatása** metódus megvalósításában a a **RoleEntryPoint** osztály tartalmaz egy hívása **Thread.Sleep(Timeout.Infinite)** , hogy az biztosítsa a szerepkör életben határozatlan ideig. Felülbírálásakor a **futtatása** (amely általában a háttér feladatok végrehajtásához szükséges) módszer, engedélyeznie kell a nem a kódot, a kilépéshez a metódusból, kivéve, ha szeretné újrahasznosítani a szerepkör példánya.
* Egy tipikus megvalósításában a **futtatása** metódus a háttérben futó feladatot, és egy hurok szerkezet, amely rendszeresen ellenőrzi a háttér-feladatok állapotának elindításához kódot tartalmazza. A sikertelen vagy cancellation jogkivonatokat, amelyek jelzik, hogy a feladatok már befejeződtek figyelő újraindítható.
* Háttérfeladat nem kezelt kivételt jelez, ha ezt a feladatot kell újrahasznosított miközben lehetővé teszi bármely más háttérfeladatok is működőképes marad, a szerepkörben. Azonban ha a kivétel okozta sérülése, objektumok, például a megosztott tároló, a feladat kívüli kivétel kezelje a **RoleEntryPoint** osztály, minden feladat megszakítása kell, és a **futtatása**metódus engedélyezni kell végződnie. Azure indul újra a szerepkört.
* Használja a **OnStop** módszert, szüneteltetése vagy háttérfeladatok kill és erőforrások törlése. Ez magában, előfordulhat, hogy hosszú futású vagy többlépéses feladat leállítása. Nagyon fontos figyelembe venni, hogyan ehhez adatok bekövetkező inkonzisztencia elkerülése érdekében. Ha a szerepkör példánya csak egy felhasználó által kezdeményezett leállítást, a kódot futtató bármilyen okból leállítja a **OnStop** metódus Kényszerített megszakítás előtt öt percen belül el kell végezni. Győződjön meg arról, hogy a kód is elvégezhető, hogy időben tűri befejezéséig nem fut.  
* Az Azure load balancer kezdődik irányítja a forgalmat a szerepkör-példány esetén a **RoleEntryPoint.OnStart** metódus értékét adja vissza **igaz**. Ezért, fontolja meg az inicializálási kód helyezésének a **OnStart** metódust, hogy sikeresen inicializálni fogja szerepkör-példányok nem fogadni a forgalmat.
* Indítási feladatok mellett a módszereket is használhatja a **RoleEntryPoint** osztály. Indítási feladatok bármely szükséges beállításokat, az az Azure load balancer módosítható, mert ezeket a feladatokat hajtja végre a szerepkör bármely kéréseket fogad előtt inicializálni kell használnia. További információkért lásd: [indítási feladatok futtatása az Azure-ban](/azure/cloud-services/cloud-services-startup-tasks/).
* Hiba található egy indítási tevékenységhez, ha azt a szerepet folyamatos újraindítását válthatja ki. Ezzel megakadályozhatja, egy virtuális IP-cím (VIP) cím felcserélés egy korábban előkészített verzióról hajt végre, mert a lapozófájl-kapacitás a szerepkör kizárólagos hozzáférésre van szüksége. Ez nem olvashatók be a szerepkör újraindítása közben. A probléma megoldásához:
  
  * Adja hozzá a következő kódot elejéhez a **OnStart** és **futtatása** a szerepkörben módszerek:
    
    ```C#
    var freeze = CloudConfigurationManager.GetSetting("Freeze");
    if (freeze != null)
    {
     if (Boolean.Parse(freeze))
       {
         Thread.Sleep(System.Threading.Timeout.Infinite);
     }
    }
    ```
    
  * Adja hozzá a definícióját a **rögzítése** beállítását egy logikai értéket a ServiceDefinition.csdef és ServiceConfiguration.\*. a szerepkör a szolgáltatáskonfigurációs séma fájlok létrehozásáról és **hamis**. Ha a szerepkör egy ismételt újraindítási módba kerül, módosíthatja a beállításokat, hogy **igaz** szerepkör-végrehajtási rögzítése, és engedélyezze a korábbi verziójával kell cserélni.

#### <a name="more-information"></a>További információ
* [Számítási erőforrás-összevonási minta](http://msdn.microsoft.com/library/dn589778.aspx)
* [Ismerkedés az Azure WebJobs SDK-val](/azure/app-service-web/websites-dotnet-webjobs-sdk-get-started/)


## <a name="partitioning"></a>Particionálás
Ha úgy dönt, hogy a háttér-feladatok belül (például egy webalkalmazás, webes szerepkör, meglévő feldolgozói szerepkör vagy virtuális gép) meglévő számítási példány is, meg kell fontolnia, hogy ez milyen hatással a számítási példány és a háttérfeladat magát minőségi jellemzőit. Ezek a tényezők segít határozza meg, hogy tartozó közös elhelyezése a tevékenységeket, amelyek a meglévő számítási példányt vagy egy különálló számítási példány külön őket:

* **Rendelkezésre állási**: háttérfeladatok előfordulhat, hogy nincs szükség a rendelkezésre állás mint az alkalmazás egyéb részeinek azonos szinten különösen a felhasználói felület és más, amely felhasználói beavatkozást közvetlenül érint. Háttérfeladatok előfordulhat, hogy jobban tűrik a késés, a rendszer kapcsolódási hibák, és más tényezők adott hatása rendelkezésre állási, mert a művelet a várólistára tehető. Azonban lehetnek olyan elegendő kapacitása megakadályozza, hogy sikerült várólisták blokkolása és érinti az egész biztonsági mentését.
* **Méretezhetőség**: háttérfeladatok valószínűleg egy különböző méretezhetőség követelmény, mint a felhasználói felület és az alkalmazás interaktív részei. A felhasználói felület skálázás csúcsait igény, amíg a függőben lévő háttérfeladatok előfordulhat, hogy teljesíteni kevésbé foglalt időpontokat során egy kevesebb teljesítéséhez szükséges lehet a számítási példányokért száma.
* **Rugalmasság**: sikertelen volt-e a számítási-példány csak háttérfeladatok nem gyermekekéhez hatással lehetnek az alkalmazás egészének Ha ezeket a feladatokat a kérelem várólistára és Elhalasztott, amíg a feladat be nem érhető el újra. A számítási példány és/vagy a feladatokat egy megfelelő intervallumon belül indítható újra, ha az alkalmazás felhasználóinak előfordulhat, hogy nem érinti.
* **Biztonsági**: Előfordulhat, hogy háttérfeladatok különböző biztonsági követelményeket vagy korlátozásokat, mint a felhasználói felület vagy az alkalmazás részei. Különálló számítási példánnyal, adjon meg egy eltérő biztonsági környezetben, a feladatok. Például forgalomirányító minták használatával elkülöníteni a felhasználói felületen háttér számítási példányokért biztonsági és elválasztási maximalizálása érdekében.
* **Teljesítmény**: akkor válassza ki a számítási példány háttér feladatok kifejezetten egyezés a feladatok teljesítménybeli követelményei. Ez azt jelentheti, hogy egy kevésbé költséges számítási beállítás használatával, ha a feladatok van szükség ugyanahhoz a feldolgozási funkciókkal rendelkeznek, mint a felhasználói felület, vagy egy nagyobb példány, ha további kapacitást és erőforrásokat igényelnek.
* **Kezelhetőségi**: háttérfeladatok lehet egy másik fejlesztésére és üzembe ritmust az fő alkalmazáskód vagy a felhasználói felületen. Központi telepítés egy külön számítási példányhoz leegyszerűsítheti a frissítések és versioning.
* **Költség**: hozzáadása számítási hajtható végre a háttérben tevékenységek növekedése üzemeltetési költségek-példányokat. Alaposan gondolja át a kompromisszum további kapacitást és a további költségek között.

További információkért lásd: [vezető választás mintát](http://msdn.microsoft.com/library/dn568104.aspx) és [versengő fogyasztók mintát](http://msdn.microsoft.com/library/dn568101.aspx).

## <a name="conflicts"></a>Ütközések
A háttérben futó feladat több példánya van, akkor lehetséges, hogy fog "versenyeznek" erőforrások és szolgáltatások, például adatbázisok és a tároló elérésére. Az egyidejű hozzáférés ütközéseket okozhat a szolgáltatások rendelkezésre állásának és az adatok integritásának Erőforrásverseny eredményezhet. Erőforrásverseny tudja oldani a pesszimista zárolási megközelítést. Ez megakadályozza, hogy a versengő egyidejűleg szolgáltatások elérésére, vagy adatokat rongál feladat példányai.

Az ütközések feloldása egy másik módszer célja egypéldányosként, háttérfeladatok meghatározása, hogy valaha is csak egy futó példány. Azonban ez megszünteti az a megbízhatóságát és teljesítményét előnyöket, amely egy több-példány konfigurációja nyújt. Ez különösen igaz, ha a felhasználói felület megadhatja a megfelelő munkahelyi egynél több háttérfeladat foglalt tartani.

Győződjön meg arról, hogy a háttérben futó feladat automatikusan újraindíthatja és arról, hogy rendelkezik-e elegendő kapacitással a kereslet csúcsait folyamatosan növekvő adatmennyiségnek elengedhetetlen. Ehhez a számítási példány elegendő erőforrással rendelkező lefoglalása egy várólista mechanizmus, amely képes tárolni az igény szerinti csökkenése esetén újabb végrehajtási kérelmek implementálásával, vagy az alábbi eljárások együttes használatával.

## <a name="coordination"></a>Problémakoordinálás
A háttérben futó feladatot lehet, hogy bonyolult, és előfordulhat, hogy több egyedi feladatokat futtatni kívánt eredmény elérése érdekében vagy a követelmények teljesítéséhez. Esetében gyakori, a feladat felosztása kisebb leplezett lépésekre vagy több fogyasztó végrehajtható résztevékenység forgatókönyvekben. Többlépéses feladatok lehet hatékonyabb és rugalmasabb, mivel előfordulhat, hogy az egyes lépéseket újrafelhasználható több feladatok. A rendszer is könnyen hozzáadása, eltávolítása vagy módosítása a lépések sorrendjét.

Több feladatok és lépések koordinációs kihívást jelenthet, de három közös mintáról olvashat, amelyek segítségével a megoldás megvalósítását Útmutató:

* **Egy feladat decomposing több újrafelhasználható lépést a**. Az alkalmazás akkor lehet szükség, amely feldolgozza az adatokat a különböző, eltérően összetett különböző feladatok végrehajtásához. Lehet, hogy egy egyszerű, de rugalmatlan megközelítése az alkalmazás végrehajtási egységes modulként a feldolgozás elvégzéséhez. Azonban ez a megközelítés valószínű, hogy csökkentse a lehetőségek a kód újrabontása, optimalizálása, vagy ha azonos feldolgozása részei szükség az alkalmazáson belül máshol újbóli felhasználása. További információkért lásd: [pipe-ok és a szűrők mintát](http://msdn.microsoft.com/library/dn568100.aspx).
* **A lépéseket egy feladat végrehajtásának kezelése**. Egy alkalmazás előfordulhat, hogy feladatokat, amelyek több lépést (ezek közül néhány lehet, hogy távoli szolgáltatások invoke vagy távoli erőforrásokhoz férnek hozzá) tartalmazza. Előfordulhat, hogy az egyes lépéseket egymástól független, de azok vannak összehangolva. az alkalmazás logika, amely megvalósítja a feladatot. További információkért lásd: [Feladatütemező ügynök felügyelő mintát](http://msdn.microsoft.com/library/dn589780.aspx).
* **A sikertelen helyreállítási feladat kezelése lépések**. Egy alkalmazás előfordulhat, hogy vissza kell vonnia a munkát, (amelyek együtt határozza meg, hogy idővel konzisztenssé művelet) lépések egy sorozatát által végrehajtott műveletek egy, vagy további lépések sikertelenek. További információkért lásd: [Compensating tranzakció mintát](http://msdn.microsoft.com/library/dn589804.aspx).


## <a name="resiliency-considerations"></a>Rugalmasság kapcsolatos szempontok
Háttérfeladatok rugalmas ahhoz, hogy az alkalmazás megbízható szolgáltatásokat kell lennie. Amikor tervezése és kialakítása háttérfeladatok, vegye figyelembe a következő szempontokat:

* Háttér-feladatok kezelésére szerepkör vagy szolgáltatás újraindítása nélkül rongál adatok, vagy ellentmondás bevezetni az alkalmazás képesnek kell lennie. Hosszan futó vagy többlépéses feladatok, érdemes lehet *mutató ellenőrizze* mentésével feladatok állapotának állandó tároló, vagy a várólistán lévő üzenetek Ha ez szükséges. Például megőrizni az állapotadatokat, egy üzenet a várólistában lévő, és fokozatosan frissítse az állapotadatokat a tevékenység végrehajtása az, hogy a feladat tudja feldolgozni a legutóbbi ellenőrzőponttól jó--az elejétől újraindítása helyett. Azure Service Bus-üzenetsorok használata esetén az azonos terület engedélyezéséhez üzenet munkamenetek használatával. Munkamenetek mentéséhez és a kérelem feldolgozása állapotának beolvasására használatával teszik a [SetState](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate?view=azureservicebus-4.0.0) és [GetState](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession.getstate?view=azureservicebus-4.0.0) módszerek. Megbízható többlépéses folyamatok és munkafolyamatok megtervezésével kapcsolatos további információkért lásd: [Feladatütemező ügynök felügyelő mintát](http://msdn.microsoft.com/library/dn589780.aspx).
* Amikor webes vagy feldolgozói szerepköröket használ több háttérfeladatok futtatásához, a felülbírálásának tervezése a **futtatása** metódus sikertelen vagy memóriavesztésének előfordulási helyét feladatok figyelése, és újra kell indítania őket. Ha ez nem gyakorlati, és a feldolgozói szerepkör használ, a feldolgozói szerepkör szerint kilépésekor kényszerítheti a **futtatása** metódust.
* Használatakor a várólisták kommunikálni háttérfeladatok, a várólisták működhet-e a feladatokat az alkalmazás során küldött kérelmek tárolására puffer magasabb, mint a szokásos terhelés alatt áll. Ez lehetővé teszi, hogy a feladatokat a felhasználói felületen kevésbé forgalmas időszakokban szinkronizálásához. Azt is jelenti, hogy a szerepkör újrahasznosítási nem akadályozza meg a felhasználói felületen. További információkért lásd: [terhelés simítás mintát várólista alapú](http://msdn.microsoft.com/library/dn589783.aspx). Ha egyes feladatok fontosabbak, mint a többire, vegye fontolóra a [prioritású várólista mintát](http://msdn.microsoft.com/library/dn589794.aspx) annak érdekében, hogy ezek a feladatok futtatása kevésbé fontos néhányat a meglévők közül.
* Háttérfeladatok üzenetek által kezdeményezett, illetve üzenetek feldolgozásához kell megtervezni, inkonzisztenciát kezelni, például a sorrendje, nem érkező üzenetek ismételten üzeneteket, amelyek hibát jelez (más néven *elhalt üzenetek*), és üzenetek kézbesítési egynél többször. A következőket ajánljuk figyelmébe:
  * Meghatározott sorrendben, például azokkal, amelyek módosítani az adatokat (például egy értéket ad hozzá egy meglévő érték) a meglévő adatok érték alapján fel kell dolgozni, előfordulhat, hogy nem érkeznek, az elküldés volt eredeti üzenetek. Azt is megteheti akkor előfordulhat, hogy kezelje, minden példány különböző terhelése miatt más sorrendben háttérfeladat különböző példányai. Meghatározott sorrendben kell végrehajtani üzenetek tartalmaznia kell a sorszámot, kulcs vagy valamilyen más mutató, háttérfeladatok segítségével győződjön meg arról, hogy a megfelelő sorrendben feldolgozásra kerülnek. Azure Service Bus használ, ha üzenet munkamenetek segítségével garantálja a szállítási sorrendjét. Azt azonban általában hatékonyabb, ha lehetséges, a folyamat kialakítani, hogy az üzenet sorrendje nem lényeges.
  * Háttérfeladat általában üzenetet a várólistában, amely ideiglenesen elrejti azokat más üzenet fogyasztói betekintés. Törli az üzenetek sikeresen feldolgozása után. Ha háttérfeladat nem sikerül, egy üzenet feldolgozása közben, az üzenet újból meg fog jelenni a várólistában a betekintés idő lejárta után. A feladat, vagy erre a következő feldolgozási ciklusban másik példánya kerül feldolgozásra. Az üzenet a fogyasztói következetesen hibát okoz, ha blokkolja a feladat, a várólista, és végül magának az alkalmazásnak, amikor a várólista megtelik. Ezért elengedhetetlen észlelni és eltávolítani az elhalt üzenetek az üzenetsorból. Ha az Azure Service Bus használ, hibát okozó üzenetek helyezheti át automatikusan vagy manuálisan egy társított halott levél várósor.
  * Várólisták garantált *legalább egyszer* kézbesítési mechanizmus, de előfordulhat, hogy az azonos üzenetet kézbesíteni egynél többször. Ezenkívül ha háttérfeladat nem sikerül, egy üzenet feldolgozása után, de a várólista törlése előtt, az üzenet is elérhető lesz újra feldolgozásra. Háttérfeladatok idempotent, ami azt jelenti, hogy ugyanaz az üzenet feldolgozása egynél többször nem okozhat hiba vagy ellentmondás az alkalmazásadatok kell lennie. Egyes műveletek esetében természetesen az idempotent, például a tárolt érték beállítása a megadott új értékre. Azonban műveletek, például egy érték hozzáadása egy meglévő tárolt értéket annak ellenőrzése, hogy a tárolt érték továbbra is ugyanaz, mint amikor az eredetileg üzenet nélkül inkonzisztenciát okozhat. Az Azure Service Bus-üzenetsorok beállítható úgy, hogy automatikusan eltávolítja a duplikált üzenetek.
  * Néhány üzenetküldő rendszereket használnak, például az Azure storage várólisták és az Azure Service Bus-üzenetsorok, támogatja a de-queue count tulajdonság, amely jelzi a szám, ahányszor egy üzenetet az üzenetsorból beolvasva. Ez lehet hasznos, ismétlődő és az elhalt üzenetek kezelésének. További információkért lásd: [aszinkron üzenetkezelési ismertetése](http://msdn.microsoft.com/library/dn589781.aspx) és [idempotencia minták](http://blog.jonathanoliver.com/2010/04/idempotency-patterns/).

## <a name="scaling-and-performance-considerations"></a>Méretezés és teljesítmény kapcsolatos szempontok
Háttérfeladatok kell nyújtania elegendő teljesítményét annak ellenőrzéséhez, nem az alkalmazás letiltása, és késleltetett művelet miatt inkonzisztenciát okozhat, ha a rendszer terhelés alatt van. Általában a háttérben futó feladatot futtató számítási példányokért skálázással a teljesítmény növelése. Tervezése és kialakítása háttérfeladatok, vegye figyelembe a következő szempontokat méretezhetőségével és teljesítménye:

* Azure támogatja az automatikus skálázást (kiterjesztése és a méretezés) aktuális igény szerint és a betöltés--alapján vagy előre definiált ütemezés szerint, Web Apps, Felhőszolgáltatások webes és feldolgozói szerepkörök és virtuális gépek központi telepítések. E szolgáltatás használatával ellenőrizze, hogy rendelkezik-e az alkalmazás egészének elegendő teljesítménybeli képességek ugyanakkor minimalizálja a futásidejű költségeket.
* Ahol háttérfeladatok egy más-más teljesítménybeli képesség egyéb részeitől Cloud Services-alkalmazás (például a felhasználói felületen vagy összetevők, például az adatelérési réteg), a háttér feladatokat együtt egy külön feldolgozói szerepkört üzemeltető lehetővé teszi, hogy a felhasználói felület és háttér-feladat szerepkörök méretezését a terhelés kezelésére. Ha több háttérfeladatok jelentősen más-más teljesítménybeli képességek egymástól, vegye figyelembe a külön feldolgozói szerepkörök történő osztásával őket, és egymástól függetlenül skálázás minden felhasználóiszerepkör-típus. Vegye figyelembe azonban, hogy ez növeli a futásidejű költségeket feladataival egyesítésével az kevesebb szerepkörök képest.
* A szerepkörök egyszerűen skálázás nem feltétlenül elégségesek teljesítmény terhelés adatvesztés elkerülése érdekében. Szükség lehet méretezni tárüzenetsort és egyéb erőforrások egyetlen pont, a teljes megelőzése érdekében a szűk keresztmetszetek váljon lánc feldolgozása. Figyelembe venni, az egyéb korlátozások is érvényesek, mint a maximális átviteli sebesség és egyéb szolgáltatások támaszkodó, az alkalmazás és a háttérben futó feladatot.
* Háttérfeladatok méretezéshez kell megtervezni. Például, képesnek kell lenniük dinamikusan észleléséhez használja ahhoz, hogy a figyelje, illetve üzeneteket küldeni a megfelelő várólistát tárolási sorok száma.
* Alapértelmezés szerint a webjobs-feladatok mérete, a társított Azure Web Apps példánya. Azonban ha azt szeretné, hogy csak egy példányban fiókként szándékozik futtatni webjobs-feladat, létrehozhat egy JSON-adatokat tartalmazó Settings.job fájlt **{"is_singleton": true}**. Ez csak egy példányát a webjobs-feladat futtatásához Azure kényszeríti, még akkor is, ha a kapcsolódó webalkalmazás több példánya van. Ez akkor lehet hasznos módszer kell futnia, csak egy példányban ütemezett feladatokhoz.

## <a name="related-patterns"></a>Kapcsolódó minták
* [Aszinkron üzenetkezelési ismertetése](http://msdn.microsoft.com/library/dn589781.aspx)
* [Automatikus skálázás útmutató](http://msdn.microsoft.com/library/dn589774.aspx)
* [Tranzakció mintát Compensating](http://msdn.microsoft.com/library/dn589804.aspx)
* [Versengő fogyasztók minta](http://msdn.microsoft.com/library/dn568101.aspx)
* [Útmutatás particionálás számítási](http://msdn.microsoft.com/library/dn589773.aspx)
* [Számítási erőforrás-összevonási minta](http://msdn.microsoft.com/library/dn589778.aspx)
* [Forgalomirányító minta](http://msdn.microsoft.com/library/dn589793.aspx)
* [Vezető választás minta](http://msdn.microsoft.com/library/dn568104.aspx)
* [Adatcsatornák és a szűrők minta](http://msdn.microsoft.com/library/dn568100.aspx)
* [Prioritás várólista minta](http://msdn.microsoft.com/library/dn589794.aspx)
* [Minta simítás várólista alapú betöltése](http://msdn.microsoft.com/library/dn589783.aspx)
* [A Feladatütemező ügynök felügyelő minta](http://msdn.microsoft.com/library/dn589780.aspx)

## <a name="more-information"></a>További információ
* [Skálázás feldolgozói szerepkörök az Azure-alkalmazások](http://msdn.microsoft.com/library/hh534484.aspx#sec8)
* [Háttér-feladatok végrehajtása](http://msdn.microsoft.com/library/ff803365.aspx)
* [Az Azure szerepkör indítása életciklusának](http://blog.syntaxc4.net/post/2011/04/13/windows-azure-role-startup-life-cycle.aspx) (blogbejegyzés)
* [Azure Cloud Services szerepkör életciklus](http://channel9.msdn.com/Series/Windows-Azure-Cloud-Services-Tutorials/Windows-Azure-Cloud-Services-Role-Lifecycle) (videó)
* [Mi az Azure WebJobs SDK?](https://docs.microsoft.com/azure/app-service-web/websites-dotnet-webjobs-sdk)
* [Háttérfeladatok futtatása WebJobs-feladatokkal](https://docs.microsoft.com/azure/app-service-web/web-sites-create-web-jobs)
* [Az Azure várólisták és a Service Bus-üzenetsorok - az képest és ellentétben](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted)
* [Diagnosztika a egy felhőalapú szolgáltatás engedélyezése](https://docs.microsoft.com/azure/cloud-services/cloud-services-dotnet-diagnostics)

