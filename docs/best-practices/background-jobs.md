---
title: Útmutató a háttérfeladatokhoz
description: Útmutatás a felhasználói felülettől függetlenül futó háttérfeladatokhoz.
author: dragon119
ms.date: 11/05/2018
ms.openlocfilehash: 0c48121a0d5cff33893a8f242c70f4a275c46f73
ms.sourcegitcommit: d59e2631fb08665bc30f6b65bfc7e1b75935cbd5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/05/2018
ms.locfileid: "51021933"
---
# <a name="background-jobs"></a>Háttérfeladatok

Számos különböző típusú alkalmazásban van szükség olyan háttérfeladatokra, amelyek a felhasználói felülettől függetlenül futnak. Ilyenek lehetnek a kötegelt feladatok, az intenzív feldolgozási feladatok és a hosszú futású folyamatok, például a munkafolyamatok. A háttérfeladatok felhasználói beavatkozás nélkül hajthatók végre – az alkalmazás a feladat elindítását követően folytathatja a felhasználók interaktív kéréseinek feldolgozását. Ez segít csökkenteni az alkalmazás felhasználói felületének terhelését, ami javíthatja a rendelkezésre állást és csökkentheti az interaktív válaszidőket.

Például ha egy alkalmazás bélyegképeket hoz létre a felhasználók által feltöltött képekhez, azt megteheti háttérfeladatként is, a bélyegképeket azok elkészültekor a tárolóba mentve – anélkül, hogy a felhasználónak meg kellene várnia a folyamat befejeztét. Például egy megrendelés feladása esetén a felhasználó ugyanígy inicializálja a megrendelést feldolgozó háttér-munkafolyamatot, majd a felhasználói felületen folytathatja a webes alkalmazás böngészését. Ha a háttérben futó feladat befejeződött, az frissítheti a tárolt rendelési adatokat, és e-mailben visszaigazolhatja a megrendelést a felhasználónak.

Amikor azt mérlegeli, hogy egy adott feladat háttérfeladatként legyen-e végrehajtva, a fő szempont, hogy a feladat képes-e felhasználói beavatkozás nélkül futni, illetve anélkül, hogy a felhasználói felület megvárná a feladat befejezését. Az olyan feladatok, amelyekhez szükséges, hogy a felhasználó vagy a felület várakozzon, nem alkalmasak háttérben való futtatásra.

## <a name="types-of-background-jobs"></a>Háttérfeladatok típusai
A háttérfeladatok rendszerint a következő típusú feladatok közül egyet vagy többet tartalmaznak:

* Számításigényes feladatok, például matematikai számítások vagy szerkezeti modellek elemzése.
* Adatátvitel-igényes feladatok, például tárolási tranzakciók sorozatának végrehajtása vagy fájlok indexelése.
* Kötegelt feladatok, például éjszakai adatfrissítések vagy ütemezett feldolgozás.
* Hosszan futó munkafolyamatok, például megrendelések teljesítése vagy szolgáltatások és rendszerek üzembe helyezése.
* Bizalmas adatok feldolgozása, ahol a feladat egy biztonságosabb helyre lesz továbbítva feldolgozásra. Például előfordulhat, hogy a bizalmas adatokat nem szeretné egy adott webalkalmazáson belül feldolgozni. Ehelyett például a [Gatekeeper-minta](https://msdn.microsoft.com/library/dn589793.aspx) használatával az adatokat átadja egy elkülönített háttérfolyamatnak, amely hozzáféréssel rendelkezik egy védett tárolóhoz.

## <a name="triggers"></a>Eseményindítók
A háttérben futó feladatok számos különböző módon indíthatók el. Az alábbi kategóriák valamelyikébe tartoznak:

* [**Eseményvezérelt eseményindítók**](#event-driven-triggers). A feladat egy eseményre, általában a felhasználó által végrehajtott műveletre vagy a munkafolyamat egy lépésére válaszképpen indul.
* [**Ütemezésvezérelt eseményindítók**](#schedule-driven-triggers). A feladatot egy időzítésen alapuló ütemezés indítja el. Ez lehet egy ismétlődő ütemezés szerint vagy egy egyszeri hívás, amely egy későbbi időpontra lett megadva.

### <a name="event-driven-triggers"></a>Eseményvezérelt eseményindítók
Az eseményvezérelt hívás egy eseményindító segítségével indítja el a háttérfeladatot. Példa az eseményvezérelt eseményindítókra:

* A felhasználói felület vagy egy másik feladat egy üzenetet küld az üzenetsorba. Az üzenet egy lezajlott művelettel kapcsolatos adatokat tartalmaz, például egy felhasználó rendelését. A háttérfeladat figyeli ezt az üzenetsort, és észleli az új üzenet megérkezését. Kiolvassa az üzenetet, és az adatait használja a háttérfeladat bemeneti adataiként.
* A felhasználói felület vagy egy másik feladat ment vagy frissít egy értéket a tárolóban. A háttérfeladat figyeli a tárolót, és észleli a változásokat. Kiolvassa az adatokat, és azokat használja a háttérfeladat bemeneti adataiként.
* A felhasználói felület vagy egy másik feladat egy kérést küld egy végpontra, például egy HTTPS URI azonosítóra vagy egy webszolgáltatásként megnyitott API-ra. A háttérfeladat végrehajtásához szükséges adatokat a kérés részeként továbbítja. A végpont vagy a webszolgáltatás meghívja a háttérfeladatot, amely az adatokat használja bemenetként.

Tipikus példák az eseményvezérelten meghívható feladatokra a képfeldolgozás, a munkafolyamatok, az adatok távoli szolgáltatásoknak való küldése, az e-mailek küldése, valamint az új felhasználók létrehozása több-bérlős alkalmazásokban.

### <a name="schedule-driven-triggers"></a>Ütemezésvezérelt eseményindítók
Az ütemezésvezérelt hívás egy időzítő segítségével indítja el a háttérfeladatot. Példa az ütemezésvezérelt eseményindítókra:

* Egy, az alkalmazásban helyileg vagy az alkalmazás operációs rendszerének részeként futó időzítő rendszeres időközönként meghív egy adott háttérfeladatot.
* Egy másik alkalmazásban futó időzítő vagy egy időzítőszolgáltatás, például az Azure Scheduler rendszeres időközönként kérést küld egy API-nak vagy egy webszolgáltatásnak. Az API vagy a webszolgáltatás meghívja a háttérfeladatot.
* Egy külön folyamat vagy alkalmazás elindít egy időzítőt, amely egy adott időtartam elteltével vagy egy adott időpontban kezdeményezi a háttérfeladat meghívását.

Az ütemezésvezérelten meghívható feladatokra tipikus példák a kötegelt feldolgozási rutinok (például a kapcsolódó termékek listájának frissítése a felhasználók legutóbbi tevékenységei alapján), a rutinszerű adatfeldolgozási feladatok (például az indexek frissítése vagy a halmozott eredmények létrehozása), az adatok elemzése a napi jelentésekhez, a megőrzött adatok törlése, valamint az adatkonzisztencia ellenőrzése.

Ha egy olyan ütemezésvezérelt feladatot használ, amelynek egyetlen példányként kell futnia, figyeljen a következőkre:

* Ha átméretezi a feladatütemezőt futtató számítási példányt (például a Windows ütemezett feladatait használó virtuális gépet), a feladatütemező több példányban fog futni. Ezek a feladatnak egyszerre több példányát is elindíthatják.
* Ha a feladatok hosszabb ideig futnak, mint az ütemezett események között eltelt időtartam, a feladatütemező elindíthatja a feladat egy újabb példányát, miközben az előző még mindig fut.

## <a name="returning-results"></a>Eredmények visszaadása
A háttérfeladatok aszinkron módon futnak a felhasználói felülethez vagy a háttérfeladatot meghívó folyamathoz képest – egy külön folyamatban, sőt akár külön helyen is. Ideális esetben a háttérfeladatokat a meghívásuk után nem kell nyomon követni, és a végrehajtásuk folyamata nincs kihatással a felhasználói felületre vagy az azokat meghívó folyamatra. Ez azt jelenti, hogy a meghívó folyamat nem vár a feladatok befejezésére. Emiatt nem is képes automatikusan észlelni, ha a feladat befejeződik.

Amennyiben szükséges, hogy a háttérfeladat kommunikálja a meghívó feladatnak a végrehajtás folyamatát vagy befejeztét, erre ki kell alakítani valamilyen mechanizmust. Néhány példa:

* Állapotjelző érték beírása egy, a felhasználói felület vagy a meghívó feladat által elérhető tárolóba, így az szükség esetén figyelheti és ellenőrizheti ezt az értéket. A háttérfaladat által a meghívónak visszaadandó egyéb adatok is elhelyezhetők ugyanebben a tárhelyben.
* Egy, a felhasználói felület vagy a meghívó által figyelt válaszüzenetsor létrehozása. A háttérfeladat az állapotot vagy a befejezést jelző üzeneteket küldhet az üzenetsorra. A háttérfaladat által a meghívónak visszaadandó adatok is elhelyezhetők ezekben az üzenetekben. Az Azure Service Bus alkalmazása esetén használhatja a **ReplyTo** és a **CorrelationId** tulajdonságot ennek a képességnek az implementálásához.
* Elérhetővé tehet egy API-t vagy végpontot a háttérfeladatból, amelyet a felhasználói felület vagy a meghívó el tud érni az állapotadatok beszerzéséhez. A háttérfaladat által a meghívónak visszaadandó adatok is részét képezhetik ennek a válasznak.
* A háttérfeladat visszahívást kezdeményezhet a felhasználói felület vagy a hívó felé egy API-n keresztül, és előre meghatározott pontokon vagy a befejezéskor jelezheti az állapotát. Ez történhet helyben létrehozott eseményeken vagy egy közzétételi-előfizetési mechanizmuson keresztül. A háttérfaladat által a meghívónak visszaadandó adatok is részét képezhetik ennek a kérésnek vagy az esemény hasznos adatainak.

## <a name="hosting-environment"></a>Üzemeltetési környezet
A háttérfeladatokat futtathatja számos különböző Azure platformszolgáltatás használatával:

* [**Azure Web Apps és WebJobs**](#azure-web-apps-and-webjobs). A WebJobs használatával egy webalkalmazás kontextusában egyéni feladatokat hajthat végre számos különböző típusú szkript vagy végrehajtható program alapján.
* [**Azure Virtual Machines**](#azure-virtual-machines). Ha Windows-szolgáltatással rendelkezik, vagy a Windows Feladatütemezőt szeretné használni, általános érdemes a háttérfeladatokat egy dedikált virtuális gépen futtatnia.
* [**Azure Batch**](#azure-batch). A Batch platformszolgáltatás számításigényes munkák futtatását ütemezi virtuális gépek felügyelt gyűjteményében. Automatikusan képes méretezni a számítási erőforrásokat.
* [**Az Azure Kubernetes Service** ](#azure-kubernetes-service) (AKS). Az Azure Kubernetes Service egy felügyelt üzemeltetési környezetet kínál a Kubernetes az Azure-ban. 

Az alábbi szakaszok részletesebben ismertetik ezeket a lehetőségeket, és olyan szempontokat is tartalmaznak, amelyek alapján kiválaszthatja az Önnek megfelelő lehetőséget.

### <a name="azure-web-apps-and-webjobs"></a>Azure Web Apps és WebJobs

Az Azure WebJobs használatával egyéni feladatokat hajthat végre háttérfeladatként egy Azure Web App-webalkalmazásban. A WebJobs-feladatok a webalkalmazás kontextusában futnak, folyamatosan működő folyamatokként. A WebJobs-feladatok továbbá az Azure Scheduler eseményindító eseménye vagy külső tényezők bekövetkezése esetén is futtathatók, például a blobtárolók és az üzenetsorok változásakor. A feladatok igény szerint indíthatók és állíthatók le, és szabályosan állnak le. Ha egy folyamatosan futó WebJobs-feladat leáll, automatikusan újraindul. Az újrapróbálkozási és hibaműveletek konfigurálhatók.

WebJobs-feladatok konfigurálásakor:

* Ha azt szeretné, hogy a feladat egy eseményvezérelt eseményindítóra válaszoljon, a **Folyamatos futtatás** beállítással kell konfigurálnia. A szkript vagy a program a site/wwwroot/app_data/jobs/continuous mappában lesz tárolva.
* Ha azt szeretné, hogy a feladat egy ütemezésvezérelt eseményindítóra válaszoljon, az **Ütemezett futtatás** beállítással kell konfigurálnia. A szkript vagy a program a site/wwwroot/app_data/jobs/triggered mappában lesz tárolva.
* Ha valamely feladat konfigurálásakor az **Igény szerinti futtatás** beállítást választja, az az indításakor ugyanazt a kódot fogja végrehajtani, mint az **Ütemezett futtatás** beállítás.

Az Azure WebJobs-feladatok egy webalkalmazás elkülönített környezetében futnak. Ez azt jelenti, hogy hozzáféréssel rendelkeznek a környezeti változókhoz, és megoszthatják az információkat, például a kapcsolati sztringeket a webalkalmazással. A feladat eléri az azt futtató gép egyedi azonosítóját. Az **AzureWebJobsStorage** kapcsolati sztring az alkalmazásadatok Azure Storage-üzenetsoraihoz, -blobjaihoz és -tábláihoz biztosít hozzáférést, valamint a Service Bushoz az üzenetkezelés és a kommunikáció megvalósítása érdekében. Az **AzureWebJobsDashboard** kapcsolati sztring a feladat műveleti naplófájljaihoz biztosít hozzáférést.

Az Azure WebJobs-feladatok a következő jellemzőkkel rendelkeznek:

* **Biztonság**: A WebJobs-feladatokat a webalkalmazás üzembehelyezési hitelesítő adatai védik.
* **Támogatott fájltípusok**: A WebJobs-feladatokat parancssori szkriptek (.cmd), parancsfájlok (.bat), PowerShell-szkriptek (.ps1), Bash-héjparancsfájlok (.sh), PHP-szkriptek (.php), Python-szkriptek (.py), JavaScript-kódok (.js), valamint végrehajtható programok (.exe, .jar stb.) használatával definiálhatja.
* **Üzembe helyezés**: A szkripteket és a végrehajtható fájlokat az [Azure Portal](/azure/app-service-web/web-sites-create-web-jobs), a [Visual Studio](/azure/app-service-web/websites-dotnet-deploy-webjobs) és az [Azure WebJobs SDK](/azure/app-service/webjobs-sdk-get-started) használatával, vagy közvetlenül a következő helyekre másolva helyezheti üzembe:
  * Eseményindító által indított végrehajtás esetén: site/wwwroot/app_data/jobs/triggered/{feladat neve}
  * Folyamatos végrehajtás esetén: site/wwwroot/app_data/jobs/continuous/{feladat neve}
* **Naplózás**: A Console.Out INFO-ként van kezelve (megjelölve). A Console.Error HIBA-ként van kezelve. A monitorozási és diagnosztikai adatok az Azure Portalon keresztül érhetők el. A naplófájlok közvetlenül is letölthetők a webhelyről. Ezek a következő helyekre lesznek mentve:
  * Eseményindító által indított végrehajtás esetén: Vfs/data/jobs/triggered/feladatNeve
  * Folyamatos végrehajtás esetén: Vfs/data/jobs/continuous/feladatNeve
* **Konfiguráció**: A WebJobs-feladatok a portál, a REST API és a PowerShell használatával konfigurálhatók. A feladat szkriptjének gyökérkönyvtárába mentett, settings.job nevű konfigurációs fájllal adhatja meg a feladat konfigurációs adatait. Példa:
  * { "stopping_wait_time": 60 }
  * { "is_singleton": true }

#### <a name="considerations"></a>Megfontolandó szempontok

* Alapértelmezés szerint a WebJobs-feladatok a webalkalmazással együtt méreteződnek. Az **is_singleton** konfigurációs tulajdonság **true** (igaz) értékre állításával azonban konfigurálhatja úgy a feladatokat, hogy egyetlen példányon fussanak. Az egypéldányos WebJobs-feladatok az olyan tevékenységekhez hasznosak, amelyeket nem szeretne méretezni vagy egyidejűleg több példányban futtatni, például újraindexeléshez, adatelemzéshez és hasonló feladatokhoz.
* A feladatok a webalkalmazás teljesítményére gyakorolt hatásának minimalizálása érdekében az esetleges hosszabb futású vagy erőforrás-igényesebb WebJobs-feladatok futtatásához célszerű létrehozni egy üres Azure Web App-példányt egy új App Service-csomag keretében.

### <a name="azure-virtual-machines"></a>Azure-alapú virtuális gépek
A háttérfeladatok implementálható úgy, hogy megakadályozza, hogy az Azure Web Apps üzembe helyezve, vagy ezek a beállítások nem a legmegfelelőbbek. Tipikus példák erre a Windows-szolgáltatások, valamint a külső fejlesztők által készített segédprogramok és végrehajtható programok. További példák lehetnek az alkalmazást futtató környezettől eltérő végrehajtási környezethez írt programok. Például rendelkezhet egy olyan Unix- vagy Linux-programmal, amelyet egy Windows- vagy .NET-alkalmazásból kíván végrehajtatni. Az Azure-beli virtuális gépek esetében számos különféle operációs rendszer közül választhat, és a szolgáltatásokat és a végrehajtható fájlokat az adott virtuális gépen futtathatja.

Ha segítségre van szüksége azzal kapcsolatban, hogy mikor érdemes a Virtual Machinest választania tekintse meg [az Azure App Service, a Cloud Services és a Virtual Machines összehasonlítását](/azure/app-service-web/choose-web-site-cloud-service-vm/). A virtuális gépek beállításokkal kapcsolatos további információkért lásd: [méretek a Windows virtuális gépek az Azure-ban](/azure/virtual-machines/windows/sizes). A Virtual Machines használata esetén elérhető operációs rendszerekkel és előre elkészített rendszerképekkel kapcsolatos további információért lásd [az Azure Virtual Machines-piacteret](https://azure.microsoft.com/gallery/virtual-machines/).

Amennyiben a háttérfeladatot egy különálló virtuális gépen szeretné inicializálni, több lehetőség közül választhat:

* A feladatot végrehajtathatja igény szerint közvetlenül az alkalmazásból egy kérés a feladat által elérhetővé tett végpontra küldésével. Ez a kérés átadja a feladat által igényelt adatokat is. A feladatot a végpont hívja meg.
* Konfigurálhatja úgy a feladatot, hogy a választott operációs rendszerben elérhető feladatütemező vagy időzítő használatával ütemezetten fusson. Például a Windows rendszeren használhatja a Windows Feladatütemezőt a szkriptek és feladatok végrehajtatására. Vagy, ha a virtuális gépen telepítve van az SQL Server, használhatja az SQL Server Agentet a szkriptek és feladatok végrehajtására.
* Az Azure Scheduler használatával inicializálhatja a feladatot egy üzenet a feladat által figyelt üzenetsorhoz történő hozzáadásával, vagy egy kérés a feladat által elérhetővé tett API-ra történő küldésével.

A háttérfeladatok inicializálásával kapcsolatban lásd az [eseményindítókat](#triggers) ismertető fenti szakaszt.  

#### <a name="considerations"></a>Megfontolandó szempontok
Amikor azt fontolgatja, hogy a háttérfeladatokat egy Azure-beli virtuális gépen helyezze-e üzembe, vegye figyelembe a következő szempontokat:

* A háttérfeladatok egy külön Azure-beli virtuális gépen való futtatása rugalmasságot biztosít, és pontosan szabályozhatóvá teszi az inicializálást, a végrehajtást, az ütemezést és az erőforrás-kiosztást. Azonban ha kizárólag a háttérfeladatok futtatásához helyez üzembe egy külön virtuális gépet, az megnöveli a környezet futásidejű költségeit.
* Az Azure Portal nem biztosít eszközöket a feladatok monitorozásához és a meghiúsult feladatok automatikus újraindításához – bár az [Azure Resource Manager parancsmagjai](https://msdn.microsoft.com/library/mt125356.aspx) lehetővé teszik a virtuális gép alapvető állapotának monitorozását és a virtuális gép kezelését. A számítási csomópontok folyamatainak és szálainak kezeléséhez azonban nem állnak rendelkezésre eszközök. Általában a virtuális gépek használata esetén további erőfeszítésekre van szükség egy olyan mechanizmus megvalósításához, amely rendszerállapot-adatokat gyűjt a feladatból és a virtuális gép operációs rendszeréről. Erre megfelelő megoldást nyújthat a [System Center Azure-hoz készült felügyeleti csomagjának](https://www.microsoft.com/download/details.aspx?id=50013) használata.
* Érdemes fontolóra venni HTTP-végpontokon keresztül elérhetővé tett monitorozási mintavételezők létrehozását. Az ilyen mintavételezők kódja állapotellenőrzéseket hajthat végre, valamint működéssel kapcsolatos adatokat és statisztikákat gyűjthet – vagy összeállíthatja a hibainformációkat, és visszaküldheti egy felügyeleti alkalmazásnak. További információért lásd az [állapot végponti monitorozását végző mintát](../patterns/health-endpoint-monitoring.md).

További információkért lásd:

* [Virtuális gépek](https://azure.microsoft.com/services/virtual-machines/)
* [Azure Virtual Machines – gyakori kérdések](/azure/virtual-machines/virtual-machines-linux-classic-faq?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)

### <a name="azure-batch"></a>Azure Batch 

Amennyiben nagy teljesítményű számítási (HPC) feladatokat szeretne egyszerre több tíz, száz vagy ezer virtuális gépen párhuzamosan futtatni, mérlegelje az [Azure Batch](/azure/batch/) használatát.  

A Batch szolgáltatás kiosztja a virtuális gépeket, feladatokat rendel azokhoz, futtatja a feladatokat, és monitorozza az állapotot. A Batch a számítási feladat igényeinek megfelelően automatikusan méretezi a virtuális gépeket. A Batch emellett a feladatütemezésről is gondoskodik. Az Azure Batch a linuxos és a windowsos virtuális gépek használatát egyaránt támogatja.

#### <a name="considerations"></a>Megfontolandó szempontok 

A Batch nagyszerűen működik a belsőleg párhuzamos számítási feladatokkal. Végső soron kevesebb lépéssel hajthat végre párhuzamos számításokat, valamint futtathat [Message Passing Interface- (MPI-) alkalmazásokat](/azure/batch/batch-mpi) olyan párhuzamos feladatoknál, amelyeknél üzeneteket kell továbbítani a csomópontok között. 

Az Azure Batch-feladatok csomópontok (virtuális gépek) készletein futnak. Az egyik megközelítésben a készletek csak szükség esetén vannak lefoglalva, majd a feladat befejezése után törlődnek. Ez maximális kihasználtságot biztosít, mivel a csomópontok nincsenek üresjáratban, azonban a feladatnak meg kell várnia a csomópontok lefoglalását. Alternatív megoldásként előre is létrehozhat egy készletet. Ez a megközelítés minimálisra csökkenti a feladatok indításához szükséges időt, azonban előfordulhat, hogy egyes csomópontok tétlenül várakoznak. További információért lásd [a készletek és a számítási csomópontok élettartamát](/azure/batch/batch-api-basics#pool-and-compute-node-lifetime).

További információkért lásd:

* [Mi az Azure Batch?](/azure/batch/batch-technical-overview) 
* [Nagy léptékű párhuzamos számítási megoldások fejlesztése a Batch segítségével](/azure/batch/batch-api-basics) 
* [Batch- és HPC-megoldások nagyméretű számítási feladatokhoz](/azure/batch/batch-hpc-solutions)

### <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Az Azure Kubernetes Service (AKS) felügyeli az üzemeltetett Kubernetes környezetet, amelynek segítségével könnyedén üzembe és kezelhet tárolóalapú alkalmazásokat. 

A tárolók hasznosak lehetnek a háttérfeladatok futtatásához. Többek között a következő előnyöket kínálják: 

- A tárolók támogatják a nagy sűrűségű üzemeltetést. Az egyes háttérfeladatokat elkülönítheti külön tárolókba, és mindegyik virtuális gépre több tárolót is elhelyezhet.
- A tárolóvezénylő kezeli a belső terheléselosztást, konfigurálja a belső hálózatot és egyéb konfigurációs feladatokat végez.
- A tárolók szükség szerint indíthatók és állíthatók le. 
- Az Azure Container Registry használatával a tárolókat az Azure határain belül regisztrálhatja. Ez biztonsági, adatvédelmi és közelségi előnyökkel jár. 

#### <a name="considerations"></a>Megfontolandó szempontok

- A tárolóvezénylők kezelésének ismeretét igényli. Ez a fejlesztői és üzemeltetési csapat készségeitől függően akár problémát is okozhat.

További információkért lásd:

* [A tárolók az Azure-ban – áttekintés](https://azure.microsoft.com/overview/containers/) 
* [A privát Docker-tárolójegyzékek bemutatása](/azure/container-registry/container-registry-intro) 

## <a name="partitioning"></a>Particionálás
Ha úgy dönt, hogy a háttérfeladatokat egy meglévő számítási példányon belül, meg kell fontolnia, hogy ez hatással lesz a számítási példány és a háttérfeladatra minőségi attribútumaira. Három tényező segít eldönteni, hogy érdemes-e a feladatokat a meglévő számítási példányokkal közösen elhelyezni, vagy inkább ki kell szervezni azokat külön számítási példányokba:

* **Rendelkezésre állás**: A háttérfeladatok esetében nem feltétlenül szükséges ugyanolyan rendelkezésre állás, mint az alkalmazás egyéb részei, különösen a felhasználói felület és a felhasználói beavatkozás során közvetlenül érintett egyéb részek esetében. Mivel a műveletek várakoztathatók, a háttérfeladatok jobban elviselhetik a késéseket, a csatlakozási hibákból eredő újrapróbálkozásokat és a rendelkezésre állást befolyásoló egyéb tényezőket. Azonban elegendő kapacitással kell rendelkezni a kérések felgyülemlésének megakadályozásához, ami blokkolhatja az üzenetsorokat, és egészében érintheti az alkalmazás működését.
* **Méretezhetőség**: A háttérfeladatok nagy valószínűséggel különböző méretezhetőségi követelményekkel rendelkeznek, mint a felhasználói felület és az alkalmazás interaktív részei. A használati csúcsok esetén szükség lehet a felhasználói felület felskálázására, a függőben lévő háttérfeladatokat azonban kevésbé terhelt időszakokban kevesebb számítási példány használatával is el lehet végezni.
* **Rugalmasság**: A kizárólag háttérfeladatokat futtató számítási példányok meghibásodása nem feltétlenül vezet a teljes alkalmazás meghibásodásához, amennyiben ezeknek a feladatoknak a kérései várakoztathatók vagy elhalaszthatók, amíg a feladat újra elérhetővé nem válik. Ha a számítási példány és/vagy a feladatok egy megfelelő időintervallumon belül újraindíthatók, az alkalmazás felhasználóira ez nem lesz kihatással.
* **Biztonság**: A háttérfeladatok valószínűleg különböző biztonsági követelményekkel vagy korlátozásokkal rendelkeznek, mint a felhasználói felület vagy az alkalmazás egyéb részei. Különálló számítási példány használata esetén eltérő biztonsági környezetet határozhat meg ezekhez a feladatokhoz. A Gatekeeper és a hasonló minták használatával a háttérbeli számítási példányokat a maximális biztonság és izoláció érdekében elkülönítheti a felhasználói felülettől.
* **Teljesítmény**: A háttérfeladatok számítási példányainak típusát kiválaszthatja kifejezetten a feladatok teljesítménykövetelményeihez illeszkedően. Ez azt jelenti, hogy az olyan háttérfeladatokhoz, amelyek nem igényelnek a felhasználói felülethez hasonló feldolgozási kapacitást, egy kevésbé költséges számítási alternatíva is elégséges lehet, vagy épp ellenkezőleg, egy nagyobb példány lehet szükséges, ha további kapacitásokat és erőforrásokat igényelnek.
* **Kezelhetőség**: A háttérfeladatok esetleg más fejlesztési és üzembehelyezési léptékkel rendelkeznek, mint a fő alkalmazáskód vagy a felhasználói felület. Ezeket külön számítási példányon beüzemelve egyszerűsíthető a frissítés és a verziókövetés.
* **Költségek**: Ha a háttérfeladatokhoz külön számítási példányokat ad hozzá, azzal növekednek az üzemeltetési költségek. Alaposan gondolja át a további kapacitások és a további költségek közötti egyensúlyt.

További információkért lásd a [vezetőválasztási mintát](../patterns/leader-election.md) és a [versengő felhasználók mintáját](../patterns/competing-consumers.md).

## <a name="conflicts"></a>Ütközések
Ha valamely háttérfeladatnak több példánya létezik, lehetséges, hogy azok versenyezni fognak az erőforrások és szolgáltatások, például az adatbázisok és a tárolók eléréséréért. Az ilyen egyidejű elérés erőforrásversenyhez vezethet, ami pedig ütközéseket okozhat a szolgáltatások rendelkezésre állásában és a tárolóadatok integritásában. Az erőforrásversenyt pesszimista zárolási módszer alkalmazásával tudja feloldani. Ez megakadályozza, hogy a feladat versengő példányai egyidejűleg érhessék el a szolgáltatásokat, vagy károsíthassák az adatokat.

Az ütközések feloldásának egy másik módszere a háttérfeladatok egypéldányosként való definiálása, azaz hogy minden esetben csak egy példányban futtathatók. Azonban ez megszünteti a többpéldányos konfiguráció kínálta megbízhatósági és teljesítménybeli előnyöket. Ez különösen igaz, ha a felhasználói felület elegendő munkát képes biztosítani, így több háttérfeladatot is lefoglal.

Kritikus fontosságú, hogy gondoskodjon róla, hogy a háttérfeladat képes legyen automatikusan újraindulni, és rendelkezzen elegendő kapacitással az igénycsúcsok kielégítéséhez. Ezt úgy érheti el, ha elegendő erőforrással foglalja le a számítási példányokat, vagy ha kialakít egy olyan várakoztatási mechanizmust, amely képes a kéréseket az igény csökkenését követő későbbi végrehajtásra eltárolni, illetve ha ezt a két technikát együtt alkalmazza.

## <a name="coordination"></a>Koordináció
Előfordulhat, hogy a háttérfeladat összetett, és több egyedi feladatot kell futtatni az eredmény elérése érdekében vagy az összes követelmény teljesítéséhez. Ezekben a forgatókönyvekben gyakran szokás a feladatokat több kisebb diszkrét lépésre vagy több alfeladatra felosztani, amelyeket több fogyasztó hajthat végre. A többlépéses feladat-végrehajtás hatékonyabb és rugalmasabb tud lenni, mivel az egyes lépések több feladatban is újrafelhasználhatók. Emellett könnyen bővíthető, csökkenthető vagy módosítható a lépések sora.

Több feladat és lépés koordinálása kihívást jelenthet, létezik azonban három gyakori minta, amelyek segítségével könnyebben megvalósítható a megoldás:

* **A feladatok felbontása több újrafelhasználható lépésre**. Az alkalmazásnak esetleg több, változó összetettségű feladatot kell végrehajtania a feldolgozandó adatokon. Egy egyszerű, de rugalmatlan megközelítés az ilyen alkalmazások megvalósításához, ha ezt a feldolgozást egy monolitikus modulként hajtatjuk végre. Ez a megközelítés azonban valószínűleg csökkenti a kód átdolgozásának, optimalizálásának vagy ismételt felhasználásának lehetőségét, amennyiben ugyanilyen feldolgozás máshol is szükséges lenne az alkalmazásban. További információért lásd a [csöveket és szűrőket ismertető mintát](../patterns/pipes-and-filters.md).
* **A feladatlépések végrehajtásának felügyelete**. Egy alkalmazásnak esetleg több lépésből álló feladatokat kell végrehajtania (amelyek némelyike esetleg távoli szolgáltatásokat hív meg vagy távoli erőforrásokat ér el). Az egyes lépések egymástól függetlenek is lehetnek, de azokat a feladatot megvalósító alkalmazáslogika vezényli. További információért lásd a [feladatütemező ügynök felügyeleti mintáját](../patterns/scheduler-agent-supervisor.md).
* **A meghiúsult feladatlépések helyreállításának felügyelete**. Az alkalmazásnak esetleg vissza kell vonnia egy sor – együtt végül egy konzisztens műveletet alkotó – lépésben végrehajtott munkát, ha egy vagy több lépés meghiúsul. További információért lásd a [kompenzáló tranzakció mintáját](../patterns/compensating-transaction.md).


## <a name="resiliency-considerations"></a>Rugalmassággal kapcsolatos szempontok
A háttérfeladatoknak rugalmasnak kell lenniük, hogy megbízható szolgáltatásokat tudjanak biztosítani az alkalmazásnak. A háttérfeladatok tervezése és kialakítása során vegye figyelembe a következő szempontokat:

* A háttérfeladatok kezelésére anélkül, hogy az adatok sérülnének vagy inkonzisztencia keletkezne az alkalmazás újraindítása képesnek kell lennie. A hosszan futó vagy többlépéses feladatok esetében érdemes lehet *ellenőrzőpontokat* alkalmazni, aminek során a feladatok állapotát menti egy állandó tárolóban vagy üzenetekként egy üzenetsorban, ha az a megfelelőbb. Például megőrizheti az állapotadatokat egy üzenetben az üzenetsorban, majd fokozatosan frissítheti ezt az állapotadatot a tevékenység előrehaladtával, így hiba esetén a feladatot a legutolsó sikeres ellenőrzőponttól kell csak végrehajtani, és nem a legelejétől. Azure Service Bus-üzenetsorok használata esetén üzenet-munkamenetek használatával ugyanezt a forgatókönyvet valósíthatja meg. A munkamenetek segítségével mentheti és visszaállíthatja az alkalmazás feldolgozási állapotát, a [SetState](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate?view=azureservicebus-4.0.0) és a [GetState](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession.getstate?view=azureservicebus-4.0.0) metódus alkalmazásával. A megbízható többlépéses folyamatok és munkafolyamatok tervezésével kapcsolatos további információért lásd a [feladatütemező ügynök felügyeleti mintáját](../patterns/scheduler-agent-supervisor.md).
* Ha üzenetsorokat használ a háttérfeladatokkal folytatott kommunikációhoz, az üzenetsorok használhatók pufferként, amelyek tárolják a feladatoknak küldött kéréseket, amíg az alkalmazás a szokottnál magasabb terheléssel működik. Így a feladatok kevésbé forgalmas időszakokban utolérhetik a felhasználói felületet. Azt is jelenti, hogy újraindítása nem blokkolja a felhasználói felületen. További információért lásd az [üzenetsor-alapú terheléskiegyenlítési mintát](../patterns/queue-based-load-leveling.md). Ha egyes feladatok fontosabbak a többinél, vegye fontolóra az [elsőbbségi üzenetsor mintájának](../patterns/priority-queue.md) alkalmazását, amellyel biztosítható, hogy ezek a feladatok a kevésbé fontosak előtt fussanak.
* Az üzenetek vagy a folyamatüzenetek által inicializált háttérfeladatokat úgy kell kialakítani, hogy kezeljék az inkonzisztenciákat, például a soron kívül érkező üzeneteket, a sorozatosan hibát okozó üzeneteket (más néven *ártalmas üzeneteket*) és a többször kézbesített üzeneteket. A következőket ajánljuk figyelmébe:
  * A meghatározott sorrendben feldolgozandó üzenetek, például amelyek meglévő adatértékek alapján módosítják az adatokat (például egy érték hozzáadása egy meglévő értékhez) esetleg nem a küldés eredeti sorrendjében érkeznek meg. Vagy az is előfordulhat, hogy az egyes példányok különböző terhelése okán a háttérfeladat különböző példányai különböző sorrendben kezelik azokat. A meghatározott sorrendben feldolgozandó üzeneteknek ezért érdemes tartalmazniuk egy sorszámot, kulcsot vagy valamilyen egyéb jelölőt, amelynek segítségével a háttérfeladatok gondoskodhatnak róla, hogy a feldolgozás a megfelelő sorrendben történjen. Az Azure Service Bus használata esetén üzenet-munkamenetek használatával garantálhatja a megfelelő kézbesítési sorrendet. Általában azonban, ha ez lehetséges, hatékonyabb újratervezni a folyamatot úgy, hogy ne számítson az üzenetek sorrendje.
  * A háttérfeladatok általában betekintenek az üzenetekbe az üzenetsorban, ami ideiglenesen elrejti azokat a többi üzenetfogyasztó elől. Ezután a háttérfeladat törli az üzeneteket azok sikeres feldolgozását követően. Ha a háttérfeladat meghiúsul, miközben egy üzenetet dolgoz fel, az üzenet újból megjelenik az üzenetsorban a betekintés időtúllépésének lejártával. A feladat egy másik példánya fogja feldolgozni, vagy ugyanez a példány a következő feldolgozási ciklusban. Ha az üzenet következetesen hibát okoz a fogyasztóban, az blokkolja a feladatot, az üzenetsort és végül magát az alkalmazást is, amikor az üzenetsor megtelik. Ezért kritikus fontosságú az ártalmas üzenetek észlelése és eltávolítása az üzenetsorból. Az Azure Service Bus használata esetén a hibát okozó üzenetek automatikusan vagy manuálisan áthelyezhetők a feldolgozatlan üzenetek társított üzenetsorába.
  * Az üzenetsorok egy *legalább egyszeri* kézbesítési mechanizmust garantálnak, de akár többször is kézbesíthetik ugyanazt az üzenetet. Ezenkívül ha a háttérfeladat meghiúsul az üzenet feldolgozása után, de még mielőtt törölhetné azt az üzenetsorból, az üzenet ismét elérhető lesz feldolgozásra. A háttérfeladatoknak idempotenseknek kell lenniük, ami azt jelenti, hogy ugyanannak az üzenetnek a többszöri feldolgozása nem okozhat hibát vagy inkonzisztenciát az alkalmazás adataiban. Egyes műveletek természetükből fakadóan idempotensek, például ilyen egy tárolt érték átállítása egy megadott új értékre. Más műveletek azonban inkonzisztenciát okozhatnak, például ha egy értéket adunk hozzá egy meglévő tárolt értékhez annak ellenőrzése nélkül, hogy az változott-e az üzenet eredeti küldésének időpontjához képest. Az Azure Service Bus-üzenetsorok konfigurálhatók úgy, hogy automatikusan eltávolítsák a duplikált üzeneteket.
  * Egyes üzenetküldő rendszerek, például az Azure Storage-üzenetsorok támogatják egy üzenetkivételi számláló tulajdonság használatát, amely azon alkalmak számát jelzi, ahányszor az üzenet ki lett olvasva az üzenetsorból. Ez hasznos lehet az ismétlődő és ártalmas üzenetek kezeléséhez. További információért lásd [az aszinkron üzenetkezelés ismertetését](https://msdn.microsoft.com/library/dn589781.aspx) és az [idempotens mintákat](https://blog.jonathanoliver.com/idempotency-patterns/).

## <a name="scaling-and-performance-considerations"></a>A méretezéssel és a teljesítménnyel kapcsolatos szempontok
A háttérfeladatoknak elegendő teljesítményt kell biztosítania, hogy ne blokkolhassák az alkalmazást vagy okozhassanak inkonzisztenciát, ha a rendszer terhelése miatt késéssel működnek. Általában a teljesítmény a háttérfeladatot futtató számítási példányok skálázásával javítható. A háttérfeladatok tervezése és kialakítása során vegye figyelembe a következő szempontokat a skálázás és a teljesítmény kapcsán:

* Az Azure támogatja az automatikus skálázást (horizontális felskálázás és szűkítést) alapján aktuális igény és terhelés vagy egy előre meghatározott ütemezés szerint, a Web Apps és a virtuális gépek központi telepítések. Ennek a szolgáltatásnak a használatával biztosítható, hogy az alkalmazás egésze elegendő teljesítménybeli kapacitással rendelkezzen, ugyanakkor a futtatási költségek minimalizálhatók legyenek.
* Ahol a háttérfeladatok egy eltérő teljesítménybeli képességek más részeitől (például a felhasználói felületen vagy összetevők, például az adatelérési réteg) alkalmazások, a háttérfeladatok együtt egy külön számítási szolgáltatás lehetővé teszi, hogy a felhasználói felület és a háttér a feladatok egymástól független méretezését, a terhelés kezelése érdekében. Ha több háttérfeladatokat jelentősen eltérő teljesítménybeli kapacitásokkal rendelkezik, fontolja meg őket osztani, és egyes egymástól függetlenül skálázási. Vegye azonban figyelembe, hogy ez megnövelheti a futtatási költségek.
* A számítási erőforrások egyszerű skálázása nem feltétlenül elégséges a terhelés következtében fellépő teljesítményvesztés elkerüléséhez. Emellett a tároló-üzenetsorokat és egyéb erőforrásokat is méretezni kellhet, hogy a teljes feldolgozási lánc egyetlen pontja se válhasson szűk keresztmetszetté. Vegye figyelembe továbbá az egyéb korlátokat is, például a tároló, valamint az alkalmazás és a háttérfeladatok által használt egyéb szolgáltatások maximális teljesítményét is.
* A háttérfeladatokat méretezhetően kell kialakítani. Például képesnek kell lenniük dinamikusan észlelni a használatban lévő üzenetsorok számát, hogy a megfelelő üzenetsort figyelhessék, és maguk is a megfelelő üzenetsorba küldjék az üzeneteket.
* Alapértelmezés szerint a WebJobs-feladatok együtt méreteződnek a társított Azure Web Apps-példánnyal. Azonban ha azt szeretné, hogy a WebJobs-feladat csak egy példányban fusson, létrehozhat egy Settings.job fájlt, amely tartalmazza az **{"is_singleton": true}** JSON-adatot. Ez kényszeríti az Azure-t, hogy a WebJobs-feladatot csak egy példányban futtassa, még ha a kapcsolódó webalkalmazásnak több példánya van is. Ez hasznos módszer lehet az olyan ütemezett feladatokhoz, amelyeket csak egy példányban lehet futtatni.

## <a name="related-patterns"></a>Kapcsolódó minták
* [Kompenzáló tranzakció mintája](../patterns/compensating-transaction.md)
* [Versengő felhasználók mintája](../patterns/competing-consumers.md)
* [Compute-particionálási útmutató](https://msdn.microsoft.com/library/dn589773.aspx)
* [Gatekeeper-minta](../patterns/gatekeeper.md)
* [Vezetőválasztási minta](../patterns/leader-election.md)
* [Csövek és szűrők mintája](../patterns/pipes-and-filters.md)
* [Elsőbbségi üzenetsor mintája](../patterns/priority-queue.md)
* [Üzenetsor-alapú terheléskiegyenlítési minta](../patterns/queue-based-load-leveling.md)
* [Feladatütemező ügynök felügyeleti mintája](../patterns/scheduler-agent-supervisor.md)

