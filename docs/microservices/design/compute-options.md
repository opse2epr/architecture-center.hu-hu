# <a name="choosing-a-compute-option-for-microservices"></a>A mikroszolgáltatás-alapú számítási lehetőség kiválasztása

A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut. A mikroszolgáltatási architektúrában a két módszer különösen népszerű:

- A szolgáltatás az orchestrator által felügyelt dedikált csomópontok (virtuális gépek) futó szolgáltatásokat.
- A functions egy kiszolgáló nélküli architektúra használatával szolgáltatásként (FaaS).

Bár ezek nem az egyetlen lehetőség, mind a mikroszolgáltatások megközelítéseket. Egy alkalmazás tartalmazhat mindkét módszerénél.

## <a name="service-orchestrators"></a>Szolgáltatás vezénylők

Az orchestrator szolgáltatások használatával történő üzembe helyezéséről kapcsolódó feladatokat végzi. Ezek a feladatok közé tartozik például a szolgáltatások csomópontok,-szolgáltatások újraindítása a nem megfelelő állapotú szolgáltatások állapotának figyelése, terheléselosztási hálózati forgalom elosztását a szolgáltatáspéldány, a szolgáltatásészlelés, a szolgáltatáspéldányok számának méretezése és alkalmazása konfiguráció frissítéseit. Népszerű szervezőeszközökkel helyezheti őket a Kubernetes, a Service Fabric, DC/OS és Docker Swarm közé tartozik.

Az Azure platformon vegye figyelembe a következő beállításokat:

- [Az Azure Kubernetes Service](/azure/aks/) (AKS) egy olyan felügyelt Kubernetes szolgáltatás. Az AKS rendelkezések Kubernetes és közzéteszi a Kubernetes API-végpontokat, de üzemelteti, és felügyeli a Kubernetes vezérlősík hajt végre automatikus frissítésekre és automatikus javításokat, automatikus skálázást és más felügyeleti feladatokat. Is felfoghatók AKS, hogy a "Kubernetes API-k szolgáltatás."

- [A Service Fabric](/azure/service-fabric/) csomagolása, üzembe helyezése és kezelése a mikroszolgáltatások egy elosztott rendszerplatform. Mikroszolgáltatások Service Fabric telepíthetők, tárolók, bináris végrehajtható fájlok, vagy mint [Reliable Services](/azure/service-fabric/service-fabric-reliable-services-introduction). A Reliable Services programozási modell révén szolgáltatások közvetlenül a Service Fabric programozási API-kat a rendszer, a jelentés állapotának lekérdezése, konfigurálásról és a kód módosításait az értesítések fogadásához és egyéb szolgáltatások észlelését. A Service Fabric legfontosabb különbséget az állapotalapú szolgáltatások használatával erős koncentrálhat [a Reliable Collections](/azure/service-fabric/service-fabric-reliable-services-reliable-collections).

- [Az Azure Container Service](/azure/container-service/) (ACS) egy Azure-szolgáltatás, amely lehetővé teszi egy üzemkész DC/OS, Docker Swarm vagy Kubernetes-fürt üzembe helyezése.

  > [!NOTE]
  > Bár az ACS Kubernetes támogatja, ajánlott az AKS segítségével Kubernetes az Azure-ban való futtatásához. Az AKS továbbfejlesztett felügyeleti funkciók és a költséget biztosít.

## <a name="containers"></a>Containers

Más személyek beszélni tárolók és mikroszolgáltatások ugyanaz, mintha. Bár ez nem igaz &mdash; nem szükséges tárolók, mikroszolgáltatások &mdash; tárolók rendelkezik, amely különösen a mikroszolgáltatások, mint például előnyöket:

- **Hordozhatóság**. Tárolórendszerkép csomag egy különálló futtató anélkül, hogy kódtárakat vagy egyéb függőségek telepítése. Így azok egyszerűen üzembe helyezhetők. Tárolók indíthatók és úgy regisztrálhat új példányokat további terhelés kezeléséhez, illetve a csomóponthibák gyorsan, leállt.

- **Sűrűségű**. Tárolók olyan egyszerűsített képest a futó virtuális gép, mert ugyanazt az operációs rendszer-erőforrásokat. Amely lehetővé teszi, hogy több tároló telepítését egyetlen csomópont, amely akkor különösen hasznos, ha az alkalmazást több kis szolgáltatásra áll.

- **Erőforrás-elkülönítést**. Korlátozhatja a mennyi memóriát és CPU, amely elérhető egy tárolóba, amely segít annak biztosításában, hogy egy elszabadult folyamat nem lefoglalhat gazdagép-erőforrásokat. Tekintse meg a [válaszfal minta](../../patterns/bulkhead.md) további információt.

## <a name="serverless-functions-as-a-service"></a>Kiszolgáló nélküli (Functions szolgáltatás)

Az egy [kiszolgáló nélküli](https://azure.microsoft.com/solutions/serverless/) architektúra, nem kell felügyelnie a virtuális gépek vagy a virtuális hálózati infrastruktúra. Ehelyett kell telepíteni, kód és az üzemeltetési szolgáltatás kezeli a kód üzembe egy virtuális Gépet az alakzatot, és futtassa a jelentést. Ez a megközelítés általában kis részletes funkciók, amelyek koordinált eseményalapú eseményindítókat használó alkalmazást. Ha például egy üzenetet egy üzenetsorba való elhelyezni kívánt is kiválthatják a függvény, amely az üzenetsorból olvas, és feldolgozza az üzenetet.

[Az Azure Functions](/azure/azure-functions/) egy kiszolgáló nélküli számítási szolgáltatás, amely támogatja a különböző függvény eseményindítók, beleértve a HTTP-kérelmekre, Service Bus-üzenetsorok és az Event Hubs-események. Teljes listáját lásd: [Azure Functions eseményindítók és kötések fogalmak](/azure/azure-functions/functions-triggers-bindings). Emellett érdemes lehet [Azure Event Grid](/azure/event-grid/), ez egy felügyelt esemény-útválasztó szolgáltatás az Azure-ban.

<!-- markdownlint-disable MD026 -->

## <a name="orchestrator-or-serverless"></a>Az orchestrator vagy a kiszolgáló nélküli?

<!-- markdownlint-enable MD026 -->

Az alábbiakban néhány szempontot figyelembe kell venni egy orchestrator-módszert és a egy kiszolgáló nélküli megközelítés közötti választáshoz.

**Kezelhetőségi** kiszolgáló nélküli alkalmazás a könnyen kezelhető, mert a platform kezeli az összes a számítási erőforrásokat az Ön számára. Az orchestrator felügyelete és a egy fürt konfigurálása bizonyos aspektusainak kivonatolja, amíg azt nem teljesen rejt el az alapul szolgáló virtuális gépeket. Az orchestrator-kell kapcsolatos problémák, például a terheléselosztás, a CPU és memória-felhasználás, és a hálózati terhelés.

**Rugalmasság és irányítás**. Az orchestrator felett konfigurálása és kezelése a szolgáltatások és a fürt nagy fokú biztosítja. Az egyensúlyt a további összetettséget. Kiszolgáló nélküli architektúrával adjon be néhány magas fokú felügyeletet mivel ezeket az adatokat emeli ki.

**Hordozhatóság**. Az itt felsorolt vezénylők (Kubernetes, DC/OS, Docker Swarm és a Service Fabric) is futtathatja a helyszíni vagy több nyilvános felhőkben.

**Alkalmazásintegráció**. Ez egy-egy kiszolgáló nélküli architektúra használatával összetett alkalmazások kihívást jelenthet. Az Azure-ban egy lehetőség [Azure Logic Apps](/azure/logic-apps/) az Azure Functions egy készletét. Erre a megközelítésre példa: [Azure Logic Apps szolgáltatással integrálható függvények létrehozása](/azure/azure-functions/functions-twitter-email).

**Költség**. Az orchestrator-fizet a virtuális gépek, amelyek a fürtön futtatják. Az egy kiszolgáló nélküli alkalmazást akkor kell fizetnie csak a tényleges számítási erőforrások felhasználásának. Mindkét esetben kell figyelembe vennie a költség, a további szolgáltatások, például a tárolás, adatbázisok és üzenetküldési szolgáltatások.

**Méretezhetőség**. Az Azure Functions automatikusan méretezi az igényeknek, a bejövő események száma alapján. Az orchestrator-horizontálisan növelje a fürtben futó szolgáltatás példányainak számát. A fürthöz további virtuális gépek hozzáadásával is méretezheti.

Referenciaimplementáció elsősorban használja a Kubernetes, de fejeződött használjuk az Azure Functions egy szolgáltatáshoz, azaz a szállítás előzményei szolgáltatás. Az Azure Functions remekül beválik, ha az adott szolgáltatáshoz, volt, mert egy eseményvezérelt munkaterhelés. A függvény meghívása egy Event Hubs-trigger használatával, a szolgáltatás szükséges minimális mennyiségű kódot. Emellett a szállítás előzményei szolgáltatás nem képezi részét a fő munkafolyamat, a Kubernetes-fürt-on kívül futnak, nem befolyásolja a felhasználó által kezdeményezett műveletek végpontok közötti késését.

## <a name="next-steps"></a>További lépések

> [!div class="nextstepaction"]
> [Szolgáltatások közötti kommunikáció](./interservice-communication.md)