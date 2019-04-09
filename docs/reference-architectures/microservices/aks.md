---
title: A Mikroszolgáltatási architektúra az Azure Kubernetes Service (AKS)
description: A mikroszolgáltatási architektúra az Azure Kubernetes Service (AKS) üzembe helyezése
author: MikeWasson
ms.date: 12/10/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 535c53faa810f74299e715a204e427c8919ce360
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/08/2019
ms.locfileid: "59069024"
---
# <a name="microservices-architecture-on-azure-kubernetes-service-aks"></a>A Mikroszolgáltatási architektúra az Azure Kubernetes Service (AKS)

A referenciaarchitektúrák a mikroszolgáltatás-alkalmazások üzembe helyezett Azure Kubernetes Service (AKS) jeleníti meg. A legtöbb telepítés esetén a kiindulási pont lehet AKS alapkonfiguráció ismerteti. Ez a cikk a Kubernetes alapszintű ismeretét feltételezi. A cikk elsősorban az infrastruktúra és a fejlesztési és üzemeltetési megfontolások az aks-en futó a mikroszolgáltatási architektúra szolgál. A mikroszolgáltatások tervezési útmutatóért lásd: [mikroszolgáltatások készítése az Azure-ban](../../microservices/index.md).

![GitHub-embléma](../../_images/github.png) egy referenciaimplementációt, a jelen architektúra érhető el az [GitHub](https://github.com/mspnp/microservices-reference-implementation).



![Az AKS a referencia-architektúra](./_images/aks.png)

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

**Azure Kubernetes Service** (AKS). AKS-felügyelt Kubernetes-fürt üzembe helyezése Azure-szolgáltatás. 

**Kubernetes-fürt**. Az AKS feladata a Kubernetes-fürt üzembe helyezéséhez és kezeléséhez a Kubernetes-főkiszolgálóhoz. Csak az ügynökcsomópontok kezelhető.

**Virtuális hálózat**. Alapértelmezés szerint az AKS üzembe helyezéséhez az ügynökcsomópontok be egy virtuális hálózatot hoz létre. Speciális forgatókönyvek esetén létrehozhat a virtuális hálózat először, többek között miként vannak konfigurálva az alhálózatokat használó helyszíni kapcsolatok és IP-címkezelést, amellyel szabályozhatja. További információkért lásd: [speciális konfigurálása az Azure Kubernetes Service (AKS) hálózat](/azure/aks/configure-advanced-networking).

**Bejövő forgalom**. Egy bejövő HTTP (S) útvonalakat a fürtön belüli szolgáltatások tesz elérhetővé. További információkért lásd: a szakasz [API-átjáró](#api-gateway) alatt.

**Külső adattárakban**. A Mikroszolgáltatások általában állapot nélküli és állapotadatokat írni a külső adattárakban, mint például az Azure SQL Database vagy a Cosmos DB.

**Az Azure Active Directory**. Az AKS használja az Azure Active Directory (Azure AD) identitás más Azure-erőforrások Azure-terheléselosztók például elindíthatók. Az Azure AD használata felhasználói hitelesítéshez ügyfélalkalmazások is ajánlott.

**Az Azure Container Registry**. Container Registry privát Docker-lemezképek vannak telepítve a fürthöz használni. Az AKS használata az Azure AD identity tároló-beállításjegyzék hitelesítheti. Vegye figyelembe, hogy az AKS nem igényel Azure Container Registrybe. Egyéb tároló-beállításjegyzékek, például a Docker Hub is használhatja.

**Az Azure folyamatok**. Folyamatok része az Azure DevOps-szolgáltatásokkal, és a futtatások automatizált buildek, tesztek és központi telepítések. Külső CI/CD megoldásokra, például a Jenkins is használhatja. 

**Helm**. Méretezhető, Helm a Kuberneteshez készült Csomagkezelő &mdash; úgy szeretné a Kubernetes-objektumokat egyetlen egységbe, amely közzétehet, üzembe helyezését, és frissítheti.

**Az Azure Monitor**. Az Azure Monitor gyűjt, és a metrikák és naplók, beleértve a platform az Azure-szolgáltatások mérőszámait a megoldás és alkalmazások telemetrikus adatait tárolja. Ezen adatok segítségével figyelheti az alkalmazást, állítsa be a riasztások és az irányítópultok és hajtsa végre a kiváltó okok elemzése a hibák. Az Azure Monitor integrálható az aks-sel való a tartományvezérlők, csomópontok, és a tárolók, valamint a tároló naplóinak és fő csomópont naplóinak begyűjtése.

## <a name="design-considerations"></a>Kialakítási szempontok

Ez a referenciaarchitektúra irányul, mikroszolgáltatás-architektúrákat, bár az ajánlott eljárásokat számos érvénybe lépnek az aks-en futó számítási feladatoktól.

### <a name="microservices"></a>Mikroszolgáltatások

A Kubernetes-szolgáltatást objektum modell mikroszolgáltatások a Kubernetesben természetes módon. A mikroszolgáltatások általában a kód lazán kapcsolt, függetlenül üzembe helyezhető egysége. Mikroszolgáltatások általában jól meghatározott API-kon keresztül kommunikálnak, és a szolgáltatásészlelés valamilyen keresztül észlelhetőek legyenek. A Kubernetes-szolgáltatást objektum ezeknek a követelményeknek megfelelő képességeket tartalmazza:

- IP-cím. A Service objektum tartalmazza a statikus belső IP-címet a podok (ReplicaSet) egy csoportjánál. Podok is létrehozott, illetve különböző pontjain áthelyezték, a szolgáltatás mindig az e belső IP-címen érhető el.

- Terheléselosztás. A szolgáltatás IP-címre küldött adatforgalom a podok terheléselosztó pedig. 

- Szolgáltatás felderítése. Szolgáltatások rendeli hozzá a Kubernetes DNS-szolgáltatás belső DNS-bejegyzéseket. Ez azt jelenti, hogy az API-átjáró segítségével meghívhatja a háttérszolgáltatást, a DNS-név használatával. Ugyanazt a mechanizmust a szolgáltatások közötti kommunikációhoz használható. A DNS-bejegyzések szerint vannak rendszerezve névtér, így a névterek felelnek meg, amelyet környezeteket, akkor a szolgáltatás DNS-neve lesz leképezve természetesen a doména aplikace.

Az alábbi ábrán a fogalmi kapcsolat és a podok között megjelenítése. A tényleges hozzárendelést végpont IP-címek és portok kube-proxyként, a Kubernetes hálózati proxy végzi el.

![Szolgáltatások és a podok](./_images/aks-services.png)

### <a name="api-gateway"></a>API-átjáró

Egy *API-átjáró* egy átjáró, amely a külső ügyfelek és a mikroszolgáltatások között helyezkedik el. Fordított proxy, mikroszolgáltatás-alapú ügyfelek útválasztási kérelmeit funkcionál. Emellett végrehajthatnak, például a hitelesítést, az SSL-lezárást és a sebességkorlátozás különböző általános feladatok. 

Az átjáró által biztosított funkciókat módon csoportosíthatók:

- [Átjáró útválasztási](../../patterns/gateway-routing.md): A jobb oldali háttérszolgáltatások útválasztási ügyfélkéréseket. Egyetlen végpontot biztosít az ügyfelek számára, és segít a különítse el a ügyfeleket a szolgáltatásoktól.

- [Átjáró-összesítési](../../patterns/gateway-aggregation.md): Több kérés összesítési egyetlen kérelemmé, az ügyfél és a háttérkiszolgáló közötti forgalmat csökkentése érdekében.

- [Átjáró-kiürítés](../../patterns/gateway-offloading.md). Az átjáró kiszervezheti a háttérszolgáltatásokat, például az SSL-lezárást, hitelesítés, IP-engedélyezési vagy ügyfél sebessége korlátozza (szabályozás) funkcióit.

API-átjárók olyan általános [a mikroszolgáltatások tervezési minta](https://microservices.io/patterns/apigateway.html). Ezek segítségével valósítható meg a különböző technológiák száma. A leggyakoribb megvalósítás valószínűleg egy peremhálózati útválasztó fordított proxy, például az nginx-et, a HAProxy vagy Traefik, a fürtben üzembe. 

Egyéb lehetőségek a következők:

- Az Azure Application Gateway és/vagy az Azure API-felügyeleti, amelyek mindkét felügyelt szolgáltatás, amely a fürtön kívüli találhatók. Egy Application Gateway Bejövőforgalom-vezérlőjéhez jelenleg bétaverzióban.

- Az Azure Functions-proxyk. Proxyk módosíthatja a kérelmek és válaszok és átirányíthatja a kéréseket URL-cím alapján.

A Kubernetes **bejövő** erőforrástípus kivonatolja a proxykiszolgálót egy konfigurációs beállításait. Egy bejövőforgalom-vezérlőt, amely az alapul szolgáló valósítja meg a bejövő forgalom együtt működik. Nincsenek bejövő vezérlők nginx-et, a HAProxy, a Traefik és az Application Gateway (előzetes verzió), többek között.

A bejövőforgalom-vezérlőjéhez kezeli a proxykiszolgáló konfigurálása. Gyakran összetett konfigurációs fájlokat, amelyek nem könnyű, ha még nem szakértője, így a bejövőforgalom-vezérlőjéhez-ban a processzoron absztrakciós ezek igényelnek. Emellett a Bejövőforgalom-vezérlőjéhez hozzáfér a Kubernetes API-t, így intelligens döntéseket hoz útválasztást és terheléselosztási funkciók. Ha például az Nginx bejövőforgalom-vezérlőjéhez megkerüli a kube-proxy hálózati proxy.

Másrészről Ha a beállítások szabályozni kell végrehajtani, érdemes megkerülése Ez az absztrakció, és a proxykiszolgáló manuális konfigurálása. 

Fordított proxykiszolgáló a potenciális szűk keresztmetszetet vagy rendszerkritikus meghibásodási pontot, ezért mindig a magas rendelkezésre állás érdekében legalább két replika üzembe.

### <a name="data-storage"></a>Adattárolás

A mikroszolgáltatási architektúrában a szolgáltatások kell ossza meg az adattárolás. Egyes szolgáltatások saját kell a saját személyes adatokat a rejtett adatok szolgáltatások közötti függőségek elkerülése érdekében egy külön logikai tárolási. A hiba oka nem szándékos összekapcsolását szolgáltatásokat, amelyek akkor fordulhat elő, amikor a szolgáltatások megosztása az azonos mögöttes adatsémák elkerülése érdekében. Is ha szolgáltatások kezeli a saját adattárak, használhatják a megfelelő adattároló a rájuk adott vonatkozó követelményeket. További információkért lásd: [mikroszolgáltatások tervezése: Az adatok kapcsolatos szempontok](/azure/architecture/microservices/data-considerations).

Kerülje a perzisztens adatok tárolása a helyi fürt storage szolgáltatással, mert, amely összeköt, hogy az adatok a csomópontra. Ehelyett 

- Például az Azure SQL Database vagy a Cosmos dB-ben egy külső szolgáltatás használja *vagy*

- Egy Azure-lemezek vagy az Azure Files használatával tartós kötet csatlakoztatási. Ha ugyanazt a kötetet kell oszthatóak meg több podok által az Azure Files használatának.

### <a name="namespaces"></a>Névterek

Névterek segítségével a fürtön belüli szolgáltatásokat. Minden objektum egy Kubernetes-fürt névtérhez tartozik. Alapértelmezés szerint egy új objektum létrehozásakor kerül a `default` névtér. Azonban célszerű létrehozni, amelyek a fürtben az erőforrások rendszerezésének elősegítése érdekében leíró névterek.

Első lépésként névterek megakadályozhatja az elnevezési ütközések. Ha több csapat ugyanabban a fürtben, akár több száz szilárd mikroszolgáltatás-alapú, üzembe helyezése a mikroszolgáltatások lekérdezi nehéz kezelni, ha az összes, ugyanazt a névteret. Emellett névterek teszik lehetővé:

- Erőforrás-korlátozások vonatkoznak egy adott névtéren, hogy az adott a névtérhez hozzárendelt podok teljes készletét nem haladhatja meg a névtér erőforrás kvótáját.

- A szabályzatokat a névterek szintjén, beleértve az RBAC- és biztonsági házirendjeivel.

A mikroszolgáltatási architektúra, figyelembe véve a kapcsolódó kontextusokat, a mikroszolgáltatások rendszerezése és a névterek az egyes kapcsolódó kontextusai létrehozását. Például a "Megrendelések teljesítése" korlátozott környezet kapcsolódó összes mikroszolgáltatások tehet lépnek ugyanazt a névteret. Másik lehetőségként hozzon létre egy névteret minden fejlesztői csapat.

Segédprogram-szolgáltatások helyezi a saját különálló névtérben. Például előfordulhat, hogy az Elasticsearch vagy telepít Prometheus fürtfigyelést és a Helm a tiller valóban.

## <a name="scalability-considerations"></a>Méretezési szempontok

Kubernetes támogatja a horizontális felskálázás két szinten:

- A központi telepítés számára lefoglalt podok számát skálázhatja.
- A csomópontok méretezése a fürtben a fürt számára elérhető összes számítási erőforrások növelése érdekében.

Bár a horizontálisan felskálázzuk és csomópontok manuálisan, minimálisra annak esélyét, hogy a szolgáltatások magas terhelés alatt fogy ki erőforrás lesz az automatikus skálázás, használatát javasoljuk. Az automatikus skálázási stratégia figyelembe kell vennie podok és a csomópontok. Ha csak a podok horizontális felskálázása, végül meg fogják tudni elérni a erőforráskorlátok a csomópontok. 

### <a name="pod-autoscaling"></a>Podok automatikus skálázás

A vízszintes Pod méretező (HPA) méretezi a podok megfigyelt CPU, memória, vagy egyéni metrikák alapján. Podok horizontális méretezés konfigurálásával, adja meg a cél a mérőszám (például: 70 % CPU) és a replikák minimális és maximális száma. Be kell tölteni a szolgáltatásokhoz, hogy ezek a számok.

Az automatikus skálázás mellékhatása, hogy podok előfordulhat, hogy hozható létre vagy eltávolítani kívánt gyakrabban, juthatnak horizontális felskálázást és a horizontális leskálázási. Ez hatásainak csökkentése érdekében:

- Használat készültségi mintavételek, hogy a Kubernetes értesíti, amikor egy új pod készen áll a fogadja el a forgalmat.
- Ezen pod megszakítás költségvetése egyszerre hány podok is kimaradásához egy szolgáltatásból.

### <a name="cluster-autoscaling"></a>Fürt automatikus skálázás

Fürt automatikus méretező méretezi a csomópontok számát. Podok korlátozott erőforrások miatt nem lehet ütemezni, ha a fürt automatikus méretező is üzembe helyezi a további csomópontokat.  (Megjegyzés: AKS és a fürt automatikus méretező integrációjával jelenleg előzetes verzióban érhető el.)

Mivel a HPA megvizsgálja a tényleges felhasznált erőforrásokat, vagy más metrikákkal podok futását, a fürt méretező, amelyek még nincsenek ütemezve podok csomópontok üzembe helyezése. Ezért úgy tűnik, a kért erőforrások üzembe helyezéséhez a Kubernetes pod specifikációja megadott módon. Terheléses tesztelés használatával finomhangolása ezeket az értékeket.

A virtuális gép mérete nem módosítható, ezért néhány, a fürt létrehozásakor válassza ki a megfelelő Virtuálisgép-méretet az ügynök csomópontok kezdeti kapacitástervezési hajtsa végre a fürt létrehozása után. 

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

### <a name="health-probes"></a>Állapotminták

Kubernetes podot tehetők közzé állapotadat-mintavétel két típusa határozza meg:

- Készenléti teszt: Arra utasítja a Kubernetest kérelmek fogadásához készen áll-e a pod.

- Végrehajtandó működőképességi: Arra utasítja a Kubernetest, hogy podot el kell távolítani, és egy új példány indítása.

Mintavételezők mértékegységeként, esetén érdemes egy szolgáltatást a Kubernetes működéséről. A szolgáltatásnak van egy címke választó, amely megfelel a podok (nulla vagy több) készletét. Kubernetes elosztja a forgalmat, amelyek megfelelnek a választó a podokat. Csak a sikeresen elindult, és kifogástalan állapotú podok forgalom fogadására. Ha összeomlik, egy tároló, a Kubernetes megszakítja a pod, és ütemezi helyett.

Egyes esetekben egy pod nem lehet készen áll a forgalom fogadásához, annak ellenére sikeresen elindult a pod. Előfordulhat például, inicializálási feladatokat, ahol az alkalmazás a tárolóban futó dolgot betölti a memóriába, vagy a konfigurációs adatokat olvas. Jelzi, hogy egy pod kifogástalan állapotú, de nem áll készen a forgalom fogadásához, adjon meg egy készenlét. 

Liveness mintavételek kezelni az esetet, ahol podot továbbra is fut, de nem kifogástalan, és újra kell indítani. Tegyük fel, hogy egy tároló valamilyen okból szolgálja ki a HTTP-kérések, de lefagy. A tároló nem összeomlás, de bármilyen kérelmek leállt. Egy HTTP végrehajtandó működőképességi határozza meg, ha a mintavétel nem válaszol, és, amely tájékoztatja a Kubernetest, hogy indítsa újra a pod.

Mintavételezők tervezésekor az alábbiakban néhány szempontot:

- Ha a kódot egy hosszú indítási ideje, nincs a veszélye annak, hogy a működőképesség folyamat hibát fog jelezni az indítás befejezése előtt. Ennek megelőzése érdekében használja az initialDelaySeconds beállítást, ami késlelteti a mintavétel elindítását.

- A végrehajtandó működőképességi nem segít, kivéve, ha a pod újraindítása valószínű, hogy megfelelő állapotba állítsa vissza. Használhatja a működőképesség memóriavesztés vagy váratlan holtpontok ellen, de nincs pont újrapróbálásakor, amelyet szeretne azonnal újra sikertelen podot.

- Néha készültségi mintavételek szolgálnak a függő szolgáltatások ellenőrzése. Például ha podot függőség, adatbázis, a működőképesség előfordulhat, hogy ellenőrizze az adatbázis-kapcsolat. Ez a megközelítés azonban váratlan problémák hozhat létre. Elképzelhető, hogy egy külső szolgáltatás valamilyen okból átmenetileg nem érhető el. A szolgáltatás összes távolítható el a terheléselosztás, és így létrehozása előtt kaszkádolás révén terjedő hibákat okozó összes podok is sikertelen, a végrehajtandó készenléti teszt, amely miatt. Jobb módszer, hogy megvalósítása az újrapróbálkozási kezelése a szolgáltatáson belül úgy, hogy a szolgáltatás megfelelően helyreállíthatja az átmeneti hibák.

### <a name="resource-constraints"></a>Erőforrás-korlátozások

Az erőforrás-versengés hatással lehet egy szolgáltatás rendelkezésre állását. Adja meg a for containers szolgáltatásban, korlátozott erőforrások, így egyetlen tároló nem ne terhelje tovább a fürterőforrások (memória és CPU). Nem tároló-erőforrások, például szálak vagy a hálózati kapcsolat, fontolja meg a [válaszfal minta](/azure/architecture/patterns/bulkhead) erőforrások elkülönítése.

Erőforráskvóták használata korlátozni az erőforrások teljes névtér esetén engedélyezett. Ezzel a módszerrel az előtér nem ez pedig erőforráshiányt a háttérszolgáltatások erőforrások vagy fordítva.

## <a name="security-considerations"></a>Biztonsági szempontok

### <a name="role-based-access-control-rbac"></a>Szerepköralapú hozzáférés-vezérlés (RBAC)

Kubernetes és az Azure egyaránt rendelkezik mechanizmusok szerepköralapú hozzáférés-vezérlés (RBAC):

- Az Azure RBAC az Azure-ban, beleértve az új Azure-erőforrások létrehozása erőforrások elérését szabályozza. Engedélyeket rendelhet felhasználókhoz, csoportokhoz és egyszerű szolgáltatásokat. (Egy egyszerű szolgáltatást az alkalmazások által használt biztonsági azonosítót.)

- Kubernetes RBAC szabályozza a Kubernetes API-engedélyek. Például podok létrehozása és ajánlati podok olyan műveletek engedélyezett (vagy elutasítva) RBAC-n keresztül a felhasználó számára. Kubernetes-engedélyek hozzárendelése a felhasználókhoz, hozzon létre *szerepkörök* és *szerepkör kötések*:

  - A szerepkör az egy adott névtéren belül vonatkozó engedélyek egy készletét. Engedélyek vannak megadva, az igék (beolvasása, frissítése, létrehozása és törlése) az erőforrások (podok, központi telepítések, stb.).

  - Egy RoleBinding felhasználókat és csoportokat hozzárendeli egy szerepkörhöz.

  - Emellett van egy ClusterRole objektum, amely a szerepkör hasonlít, de minden névtérben létrehozott az egész fürt vonatkozik. Felhasználók vagy csoportok hozzárendelése egy ClusterRole, hozzon létre egy ClusterRoleBinding.

AKS egyesíti e két RBAC mechanizmusok. Ha egy AKS-fürtöt hoz létre, konfigurálhatja úgy, hogy a felhasználók hitelesítéséhez az Azure AD. Ez beállítása a részletekért lásd: [integrálása az Azure Active Directory és az Azure Kubernetes Service](/azure/aks/aad-integration).

Ha ez lett konfigurálva, a Kubernetes API-t (például a kubectl) hozzáférni kívánó felhasználó be kell jelentkeznie a az Azure AD hitelesítő adataik használatával.

Alapértelmezés szerint az Azure AD-felhasználót nem érhető el a fürt rendelkezik. Hozzáférés biztosításához a fürt rendszergazdája, tekintse meg az Azure AD-felhasználók vagy csoportok RoleBindings hoz létre. Ha egy felhasználó nem rendelkezik engedélyekkel egy adott művelethez, nem fog működni.

Ha a felhasználók nem férhetnek alapértelmezés szerint, hogyan a fürt rendszergazdai rendelkezik engedéllyel a szerepkör-kötések létrehozásához az első helyen? AKS-fürt rendelkezik-e hitelesítő adatokat a Kubernetes API-t kiszolgáló meghívására szolgáló két típusú: fürt felhasználói és a fürt rendszergazdája. A fürt rendszergazdai hitelesítő adatait a fürt teljes hozzáférési jogot. Az Azure CLI-paranccsal `az aks get-credentials --admin` letölti a fürt rendszergazdai hitelesítő adatait, és a kubeconfig fájlba menti őket. A fürt rendszergazdája a kubeconfig segítségével szerepköröket és szerepkör-kötések létrehozása.

Olyan hatásosak a fürt rendszergazdai hitelesítő adatokat, mert az Azure RBAC használatával elérésének korlátozása:

- Az "Azure Kubernetes Service fürt rendszergazdai szerepkör" jogosult töltse le a fürt rendszergazdai hitelesítő adatait. Csak a fürt rendszergazdák ezt a szerepkört kell rendelni.

- Az "Azure Kubernetes Service fürt felhasználói szerepkörhöz" a fürt felhasználói hitelesítő adatok letöltése szükséges jogosultsággal rendelkezik. A nem rendszergazda felhasználók ehhez a szerepkörhöz is hozzárendelhető. Ez a szerepkör nem ad adott engedélyek a fürtben Kubernetes-erőforrások &mdash; , csak lehetővé teszi a felhasználóknak az API-kiszolgálóhoz való csatlakozáshoz. 

Az RBAC-szabályzatok (mind a Kubernetes, mind a Azure) határozza meg, ha úgy gondolja, hogy a szervezet a szerepkörökről:

- Felhasználók létrehozása vagy egy AKS-fürt törlése és töltse le a rendszergazdai hitelesítő adatait?
- A fürt felügyeletét?
- Ki is erőforrások létrehozása vagy módosítása egy adott névtéren belül?

Egy célszerű hatókörhöz Kubernetes RBAC-névteret, szerepkörök és RoleBindings, hanem ClusterRoles és ClusterRoleBindings engedélyeket.

Végül pedig a kérdést, milyen engedélyekkel az AKS-fürt létrehozása és kezelése az Azure-erőforrások, például a terheléselosztók, hálózati vagy tárolási rendelkezik. Hitelesítse magát az Azure API-k, a fürt egy Azure AD-szolgáltatásnevet használja. Ha nem ad meg egy szolgáltatásnevet, a fürt létrehozásakor, az egyik automatikusan létrejön. Viszont azt, először hozza létre a szolgáltatásnevet, és rendelje hozzá a minimális RBAC-engedélyek ajánlott biztonsági eljárás. További információkért lásd: [egyszerű szolgáltatások Azure Kubernetes Service-szel](/azure/aks/kubernetes-service-principal).

### <a name="secrets-management-and-application-credentials"></a>Titkos kódok felügyeleti és a hitelesítő adatok

Alkalmazások és szolgáltatások gyakran kell hitelesítő adatokat, amelyek lehetővé teszik külső szolgáltatásokkal, például az Azure Storage vagy az SQL-adatbázishoz való kapcsolódáshoz. A kihívás, hogy biztonságban ezeket a hitelesítő adatokat, és nem szivárogtatnak ki őket. 

Azure-erőforrások az egyik lehetőség, hogy felügyelt identitásokat használ. Egy felügyelt identitás lényege, hogy egy alkalmazás vagy szolgáltatás egy Azure AD-ben tárolt azonosítóval rendelkezik, és ezt az identitást használja az Azure-szolgáltatásokkal való hitelesítéshez. Az alkalmazás vagy szolgáltatás rendelkezik egy egyszerű szolgáltatást az Azure ad-ben készült, és az OAuth 2.0 jogkivonat hitelesíti. A végrehajtó folyamat meghívja a localhost címet beszerezni a jogkivonatot. Ezzel a módszerrel nem kell minden olyan jelszavakat és a kapcsolati karakterláncokat tárolja. Használhatja a felügyelt identitások az aks-ben identitások rendel az egyes podok használatával a [aad-pod-identity](https://github.com/Azure/aad-pod-identity) projekt.

Jelenleg nem minden Azure-szolgáltatásokat támogatja a hitelesítés felügyelt identitások használatával. Egy listát lásd: [Azure-szolgáltatások, hogy a támogatás az Azure AD-hitelesítés](/azure/active-directory/managed-identities-azure-resources/services-support-msi).

Még a felügyelt identitások, akkor valószínűleg kell egyes hitelesítő adatokat vagy más alkalmazás titkainak tárolásához az Azure-szolgáltatások, amelyek nem támogatják a felügyelt identitásokból, harmadik féltől származó szolgáltatásokkal, API-kulcsok és így tovább. Íme néhány lehetőség a titkos kulcsok biztonságos tárolása:

- Az Azure Key Vault. Az aks-ben akkor is csatlakoztathatja egy vagy több titkos kulcsok a Key Vaultból kötetként. A kötet a titkos kulcsok a Key Vault olvassa be. A pod el tudja olvasni a titkos kulcsok, csakúgy, mint a rendszeres kötet. További információkért lásd: a [Kubernetes-KeyVault-FlexVolume](https://github.com/Azure/kubernetes-keyvault-flexvol) projektet a Githubon.

    A pod hitelesíti magát vagy egy pod identitással (fentebb leírt) vagy egy Azure AD-szolgáltatásnév és a egy ügyfélkulcsot használatával. Pod identitások használata ajánlott, mert a titkos ügyfélkulcsot ebben az esetben nincs szükség. 

- HashiCorp Vault. Kubernetes-alkalmazások az Azure ad-ben felügyelt identitások használatával HashiCorp Vault hitelesítheti. Lásd: [HashiCorp Vault ad elő az Azure Active Directory](https://open.microsoft.com/2018/04/10/scaling-tips-hashicorp-vault-azure-active-directory/). Vault helyezhet üzembe Kubernetes, de azt javasoljuk, futtatásához egy külön dedikált fürtben az alkalmazás-fürtből. 

- Kubernetes titkos kulcsok. Egy másik lehetőség, egyszerűen Kubernetes titkos kódok használatára. Ez a beállítás a legkönnyebben konfigurálható, de áttekinthet néhány problémát. Titkos kódok találhatók etcd, ez az elosztott kulcs-érték tároló. Az AKS [titkosítja az inaktív etcd](https://github.com/Azure/kubernetes-kms#azure-kubernetes-service-aks). A Microsoft felügyeli, a titkosítási kulcsokat.

Több előnye is van, a rendszer, például a HashiCorp tár vagy az Azure Key Vault használatával például biztosítja:

- A titkok központosított felügyeletet.
- Annak ellenőrzése, hogy az összes titkos inaktív vannak titkosítva.
- Központi kulcskezelési.
- Hozzáférés-vezérlés a titkos kulcsok.
- Naplózás

### <a name="pod-and-container-security"></a>Podok és tárolók biztonságával

Ez a lista természetesen nem teljes körű, de Íme néhány a podok és tárolók biztonságával kapcsolatos ajánlott eljárásokat: 

Védett módban nem futnak a tárolók. Védett módban hozzáférést biztosít egy tárolót a gazdagép összes eszközre. A kiemelt jogosultságú módban futó tárolók letiltása a Pod biztonsági házirendjében is megadhatja. 

Ha lehetséges, kerülje a legfelső szintű tárolókban futó folyamatok. Tárolók nem biztosítanak, biztonsági szempontból a teljes elkülönítést, így jobb egy tároló folyamatot futtató nem kiemelt jogosultságú felhasználó. 

Store kép olyan megbízható privát beállításjegyzékbe, például az Azure Container Registry vagy a Docker-beállításjegyzék megbízható. A Kubernetes érvényesítésekor már a betegfelvétel webhookok használatával győződjön meg arról, hogy podok is csak rendszerképek lekérése a megbízható beállításjegyzék.

Az ismert biztonsági rések felderítéséhez, például Twistlock és a Tengerkék, amelyek az Azure piactéren elérhető keresési megoldások használata kiszűrhető.

Automatizálhatja a lemezkép javítás ACR feladatokat, az Azure Container Registry szolgáltatás használatával. Egy tárolórendszerképet a rétegek épül. Az alap réteg tartalmazza az operációsrendszer-lemezkép és az alkalmazás keretrendszer-lemezképek, például az ASP.NET Core- vagy node.js nyelven. Az alaplemezképek általában jönnek létre az alkalmazásfejlesztők a felsőbb rétegbeli, és más projekt maintainers kell megőrizni. Ha ezek a lemezképek felső telepítve, fontos frissítését, tesztelése és ismételt üzembe helyezése saját lemezképeket, hogy minden olyan ismert biztonsági rések nem hagyja. ACR-feladatok segítségével automatizálni a folyamatot.

## <a name="deployment-cicd-considerations"></a>Üzembe helyezés (CI/CD) kapcsolatos szempontok

Az alábbiakban néhány célja a mikroszolgáltatási architektúra egy robusztus CI/CD folyamatot:

- Minden egyes csapat létrehozhat és üzembe helyezése a szolgáltatások egymástól függetlenül, tulajdonos érintő vagy más csapatok megszakítása nélkül.

- A szolgáltatás egy új verziója éles üzembe helyezés, mielőtt megtörténik fejlesztési/tesztelési/QA Environment ellenőrzés céljából. Minőségi kapuk a rendszer minden egyes fázisában érvényesíti.

- A szolgáltatás egy új verziója lehet az üzembe helyezett egymás melletti korábbi verziójával.

- Megfelelő hozzáférés-vezérlési házirendeket vannak érvényben.

- A tárolólemezképeket az éles környezetben üzembe helyezett megbízható.

### <a name="isolation-of-environments"></a>A környezetek elkülönítése

Kell több környezetet, ahol a szolgáltatások, beleértve a fejlesztési, füst tesztelése, integráció tesztelése, terheléses tesztelés díjaival, és végül éles környezetben telepít. Ezekben a környezetekben kell bizonyos fokú elkülönítés. A Kubernetes választhat, hogy fizikai elkülönítése és logikai elkülönítése között. Fizikai elkülönítése azt jelenti, hogy különálló fürt üzembe helyezése. Logikai elkülönítéssel használ, a névterek és a házirendeket, a fentebb leírt módon.

Azt javasoljuk, hogy hozzon létre egy dedikált fürt és a fejlesztési-tesztelési környezetek esetében külön fürtben. Logikai elkülönítés használatával a fejlesztési-tesztelési fürtön belüli környezeteket. A fejlesztési-tesztelési fürtre telepített szolgáltatások soha nem üzleti adatokat tartalmazó adattárakra hozzáféréssel kell rendelkeznie. 

### <a name="helm"></a>Helm

Fontolja meg a Helm használatával létrehozásának és üzembe helyezésének a szolgáltatások kezeléséhez. A Helm, amelyek segítenek a CI/CD-funkcióit tartalmazza:

- Rendszerezéséhez a Kubernetes-objektumokat az egy adott mikroszolgáltatás egyetlen Helm-diagram be.
- A diagram kubectl parancsokat helyett egy egyszeri helm parancs telepítése.
- Követési frissítések és a módosításokat, Szemantikus verziószámozást elkészíthetők visszaállítása egy korábbi verziójának használatával.
- A sablonok információkat, például a címkék és a választók, elkerülhető a tartalomkiszolgálón használatát.
- Diagramok közötti függőségek kezelése.
- Diagramok közzétételét egy Helm-adattárhoz, például az Azure Container Registrybe, és integrálja a buildelési folyamat.

További információ a Container Registry használatával, egy Helm-adattárhoz: [használata Azure Container Registry vagy a Helm az alkalmazás diagramok](/azure/container-registry/container-registry-helm-repos).

### <a name="cicd-workflow"></a>CI/CD-munkafolyamat

Mielőtt létrehozná a CI/CD munkafolyamat, ismernie kell a hogyan kódbázis fog strukturált és felügyelt.

- Működnek-e a teams, külön referenciákhoz vagy egy monorepo (egyetlen adattár)?
- Mi az az elágazási stratégia?
- Akik beküldéssel telepíthet az éles környezetbe? Egy kiadási szerepkör van?

A monorepo megközelítés hozd hónappal számolunk, de vannak előnyei és hátrányai is.

| &nbsp; | Monorepo | Több adattárakkal |
|--------|----------|----------------|
| **Előnyök** | Kód megosztása<br/>Könnyebben szabványosíthatja a kódot, és azokat az eszközöket<br/>Könnyebben újrabontása kód<br/>Könnyebben – a szabályzat egyetlen nézetben<br/> | Egyértelmű tulajdonjog csapatonként<br/>Potenciálisan kevesebb ütközések egyesítése<br/>Elválasztás álló kényszerítése segít |
| **Problémák** | Megosztott programkód módosítása hatással lehet a több mikroszolgáltatások<br/>Az egyesítési ütközések nagyobb lehetőségeit<br/>Azokat az eszközöket kell méretezhető, nagy méretű kódbázis<br/>Hozzáférés-vezérlés<br/>Összetettebb telepítési folyamat | Nehezebb lehet megosztani a kódot<br/>Nehezebb kódolási szabványok kikényszerítésére<br/>Függőségkezelés<br/>Kód alap, a gyenge felderíthetőség szórt<br/>Megosztott infrastruktúra hiánya

Ebben a szakaszban azt jelenleg egy lehetséges CI/CD-munkafolyamatot, a következő feltételek alapján:

- A kódtár monorepo mappák mikroszolgáltatás szerint vannak rendezve.
- A csapat stratégiáját az elágazási alapján [trönk-alapú fejlesztési](https://trunkbaseddevelopment.com/).
- A csapata által használt [Azure folyamatok](/azure/devops/pipelines) a CI/CD-folyamat futtatását.
- A csapata által használt [névterek](/azure/container-registry/container-registry-best-practices#repository-namespaces) az Azure Container Registry rendszerképek éles környezetben, amely továbbra is tesztelik rendszerképekből jóváhagyott elkülönítésére.

Ebben a példában egy fejlesztői dolgozik egy mikroszolgáltatás-kézbesítési szolgáltatás neve. (A név a leírt referenciaimplementációt származik [Itt](../../microservices/design/index.md#scenario).) Egy új szolgáltatás fejlesztésekor a fejlesztői egy funkciót a főágban kód ellenőrzi.

![CI/CD-munkafolyamat](./_images/aks-cicd-1.png)

Hogy a véglegesítéseket a fiókiroda tiggers a mikroszolgáltatás egy CI buildet. Szabályok szerint funkció ágak elnevezése `feature/*`. A [létrehozása csomagdefiníciós fájl](/azure/devops/pipelines/yaml-schema) tartalmaz egy eseményindítót, amely a kiadásiág-nevet és a forrás elérési útja alapján szűri. Ezt a módszert használja, minden egyes csapat a saját build folyamat lehet.

```yaml
trigger:
  batch: true
  branches:
    include:
    - master
    - feature/*

    exclude:
    - feature/experimental/*

  paths:
     include:
     - /src/shipping/delivery/
```

Ezen a ponton a munkafolyamatban a CI-build néhány csak minimális mennyiségű kódra ellenőrzés fut:

1. Kód buildelése
1. Futtatni az egységteszteket

Itt a cél, hogy tartsa a fejlesztési idő rövid, így a fejlesztő gyors visszajelzés. Ha a szolgáltatás készen áll a egyesítése a mesterrel, a fejlesztői megnyílik az új Ez elindít egy másik CI build néhány további ellenőrzést végző:

1. Kód buildelése
1. Futtatni az egységteszteket
1. A futtatókörnyezet tárolólemezkép létrehozásához
1. Futtassa a vizsgálatot a biztonsági rések a rendszerképen

![CI/CD-munkafolyamat](./_images/aks-cicd-2.png)

> [!NOTE]
> Az Azure-Adattárakkal, definiálhat [házirendek](/azure/devops/repos/git/branch-policies) ágak védelme érdekében. Például a szabályzat szükségük CI a build sikeres létrehozása és a egy jóváhagyás a jóváhagyó egyesítése a mesterrel.

Később a csapat a kézbesítési szolgáltatás új verziójának telepítése készen áll. Ehhez a kiadáskezelő hoz létre egy ágat a mintában az elnevezési mintázatot: `release/<microservice name>/<semver>`. Például: `release/delivery/v1.0.2`.
Ez elindít egy teljes CI hozhat létre, amely az előző lépések és:

1. A Docker-rendszerkép leküldése az Azure Container Registrybe. A kép a kiadásiág-nevet származó verziószámmal van megjelölve.
2. Futtatás `helm package` csomagolása a Helm-diagram
3. A Helm-csomag leküldése a Tárolójegyzékbe futtatásával `az acr helm push`.

Ha a build sikeres lesz, aktivál egy folyamatot egy Azure-folyamatok használatával [kibocsátási folyamatok](/azure/devops/pipelines/release/what-is-release-management). Ez a folyamat

1. Futtatás `helm upgrade` környezetben való üzembe helyezéséhez a Helm-diagramot a QA.
1. Jóváhagyó jelentkezik, mielőtt a csomagot az éles környezetbe helyezi át. Lásd: [kiadási központi telepítés ellenőrzési jóváhagyások használatával](/azure/devops/pipelines/release/approvals/approvals).
1. A Docker-rendszerképet az Azure Container Registry a termelési névtér újra címkékkel. Például, ha az aktuális címke `myrepo.azurecr.io/delivery:v1.0.2`, az éles címke `myrepo.azurecr.io/prod/delivery:v1.0.2`.
1. Futtatás `helm upgrade` üzembe helyezéséhez a Helm-diagram az éles környezetbe.

![CI/CD-munkafolyamat](./_images/aks-cicd-3.png)

Fontos megjegyezni, hogy egy monorepo, még akkor is, ezek a feladatok korlátozhatja az egyes mikroszolgáltatások úgy, hogy a teams telepítheti a nagy sebességű. A folyamat néhány manuális lépésből áll: Lekéréses kérelmek jóváhagyása, a kiadási ágak létrehozása és az éles fürt üzemelő jóváhagyása. Ezen lépések végrehajtása manuális házirend &mdash; azok sikerült teljesen automatizálható, ha a szervezet részesíti előnyben.
