---
title: "Jenkins-kiszolgáló futtatása az Azure-on"
description: "Ez a referenciaarchitektúra egy egyszeri bejelentkezéssel (SSO) biztosított, skálázható, nagyvállalati szintű Jenkins-kiszolgáló üzembe helyezését és üzemeltetését mutatja be az Azure-on."
author: njray
ms.date: 01/21/18
ms.openlocfilehash: 9cab4990b259695f310da339bfef3060b0905640
ms.sourcegitcommit: 3426a9c5ed937f097725c487cf3d073ae5e2a347
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/01/2018
---
# <a name="run-a-jenkins-server-on-azure"></a>Jenkins-kiszolgáló futtatása az Azure-on

Ez a referenciaarchitektúra egy egyszeri bejelentkezéssel (SSO) biztosított, skálázható, nagyvállalati szintű Jenkins-kiszolgáló üzembe helyezését és üzemeltetését mutatja be az Azure-on. Az architektúra emellett az Azure Monitor használatával monitorozza a Jenkins-kiszolgáló állapotát. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

![Az Azure-ban futó Jenkins-kiszolgáló][0]

*Töltse le az architektúra ábráját tartalmazó [Visio-fájlt](https://arch-center.azureedge.net/cdn/Jenkins-architecture.vsdx).*

Az architektúra támogatja a vészhelyreállítást az Azure-szolgáltatásokkal, több főkiszolgálót vagy állásidő nélküli magas rendelkezésre állást (HA) megvalósító fejlettebb felskálázási forgatókönyveket azonban nem tartalmaz. A különféle Azure-összetevőkkel kapcsolatos általános információkért, többek között a CI-/CD-folyamatok az Azure-on való kiépítésének részletes leírásáért lásd: [Jenkins az Azure-on][jenkins-on-azure].

Ez a dokumentum a Jenkins támogatásához szükséges alapvető Azure-műveletekkel foglalkozik. Ilyen például a build-összetevők karbantartása az Azure Storage tárolók használatával, az egyszeri bejelentkezéshez szükséges biztonsági elemek, az egyéb integrálható szolgáltatások, valamint a folyamat skálázhatósága. Az architektúra úgy van kialakítva, hogy meglévő forráskezelési adattárakkal is használható legyen. Gyakori forgatókönyv például a Jenkins-feladatok GitHub-véglegesítéseken alapuló indítása.

## <a name="architecture"></a>Architektúra

Az architektúra az alábbi összetevőkből áll:

-   **Erőforráscsoport.** Az [erőforráscsoportok][rg] Azure-objektumok csoportosítására használhatók, így élettartamuk, tulajdonosuk vagy egyéb jellemzőik alapján kezelhetők. Az erőforráscsoportok segítségével csoportosan helyezhet üzembe és monitorozhat Azure-objektumokat, a számlázási költségeket pedig erőforráscsoportonként tarthatja számon. Az erőforrásokat törölheti is készletenként. Ez nagyon hasznos tesztkörnyezetek esetében.

-   **Jenkins-kiszolgáló**. Egy, a [Jenkins][azure-market] automatizáló kiszolgálóként való futtatására üzembe helyezett virtuális gép, amely Jenkins-főkiszolgálóként szolgál. Ez a referenciaarchitektúra [az Azure-hoz készült Jenkins megoldássablonját][solution] használja, amely egy Linux (Ubuntu 16.04 LTS) rendszerű virtuális gépre van telepítve az Azure-ban. Az egyéb Jenkins-ajánlatok az Azure Marketplace áruházban érhetők el.

    > [!NOTE]
    > A virtuális gépre telepített Nginx fordított proxyként szolgál a Jenkinshez. A Nginx konfigurálásával engedélyezhető az SSL a Jenkins-kiszolgáló számára.
    > 
    > 

-   **Virtuális hálózat**. A [virtuális hálózatok][vnet] összekapcsolják az Azure-erőforrásokat, és biztosítják azok logikai elkülönítését. Ebben az architektúrában a Jenkins-kiszolgáló egy virtuális hálózatban fut.

-   **Alhálózatok**. A Jenkins-kiszolgáló egy [alhálózatba][subnet] van különítve, így a hálózati forgalom könnyebben felügyelhető és elkülöníthető anélkül, hogy ez befolyásolná a teljesítményt.

-   **NSG-k**. A [hálózati biztonsági csoportok][nsg] (NSG-k) használatával korlátozható az internetről a virtuális hálózat adott alhálózatára irányuló hálózati forgalom.

-   **Felügyelt lemezek**. A [felügyelt lemezek][managed-disk] olyan perzisztens virtuális merevlemezek (VHD), amelyek alkalmazástárolóként, valamint a Jenkins-kiszolgáló állapotának fenntartásához és vészhelyreállításhoz használhatók. Az adatlemezeket az Azure Storage tárolja. A jobb teljesítmény érdekében [Prémium szintű Storage-tárolók][premium] használata ajánlott.

-   **Azure Blob Storage**. A [Windows Azure Storage beépülő modulja][configure-storage] az Azure Blob Storage segítségével tárolja a létrehozott és más Jenkins-buildekkel megosztott build-összetevőket.

-   **Azure Active Directory (Azure AD)**. Az [Azure AD][azure-ad] támogatja a felhasználók hitelesítését, így lehetővé teszi az egyszeri bejelentkezés beállítását. Az Azure AD [egyszerű szolgáltatásai][service-principal] határozzák meg a szabályzatot és az engedélyeket a munkafolyamat egyes szerepkör-engedélyezéseihez [szerepköralapú hozzáférés-vezérlés][rbac] (RBAC) segítségével. Mindegyik egyszerű szolgáltatás társítva van egy Jenkins-feladattal.

-   **Azure Key Vault.** A titkosítást igénylő Azure-erőforrások üzembe helyezéséhez használt titkok és titkosítási kulcsok kezeléséhez ez az architektúra a [Key Vault][key-vault] szolgáltatást használja. A folyamatban foglalt alkalmazással társított titkok tárolásával kapcsolatos további segítségért lásd még a Jenkinshez készült [Azure Credentials][configure-credential] beépülő modult.

-   **Azure figyelési szolgáltatások**. Ez a szolgáltatás a Jenkinst futtató Azure-beli virtuális gépet [monitorozza][monitor]. Ez az üzemelő példány monitorozza a virtuális gép állapotát és processzorkihasználtságát, valamint riasztásokat küld ezekkel kapcsolatban.

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="azure-ad"></a>Azure AD

Az Azure-előfizetés [Azure AD][azure-ad]-bérlője segítségével engedélyezhető az egyszeri bejelentkezés a Jenkins-felhasználók számára, valamint telepíthetők az [egyszerű szolgáltatások][service-principal], amelyek biztosítják a Jenkins-feladatok hozzáférését az Azure-erőforrásokhoz.

Az egyszeri bejelentkezés hitelesítését és engedélyezését a Jenkins-kiszolgálón telepített Azure AD beépülő modul teszi lehetővé. Az egyszeri bejelentkezés használatával a felhasználók az Azure AD-ből származó vállalati hitelesítő adataikkal hitelesítők a Jenkins-kiszolgálóra való bejelentkezéskor. Az Azure AD beépülő modul konfigurálásakor megadhatja, hogy az egyes felhasználók milyen szinten férhessenek hozzá a Jenkin-kiszolgálóhoz.

A Jenkins-feladatok az Azure-erőforrásokhoz való hozzáférésének biztosításához az Azure AD-rendszergazdák egyszerű szolgáltatásokat hozhatnak létre. Ezek [hitelesített és engedélyezett hozzáférést][ad-sp] biztosítanak az alkalmazások – esetünkben a Jenkins-feladatok – számára az Azure-erőforrásokhoz.

Az [RBAC][rbac] segítségével tovább szabályozható a felhasználók vagy egyszerű szolgáltatások hozzáférése az Azure-erőforrásokhoz a hozzárendelt szerepköreik alapján. A beépített és az egyéni szerepkörök egyaránt támogatottak. A szerepkörök segítségével emellett biztonságossá tehető a folyamat, továbbá biztosítható, hogy a felhasználók vagy ügynökök felelősségi körei megfelelően legyenek kiosztva és engedélyezve. Emellett az RBAC beállításával korlátozható az Azure-objektumok elérése is. A felhasználók például kizárólag egy adott erőforráscsoport objektumainak használatára korlátozhatók.

### <a name="storage"></a>Tárolás

Az Azure Marketplace áruházból telepíthető Jenkins [Windows Azure Storage beépülő modul][storage-plugin] segítségével tárolhatók a más buildekkel és tesztekkel megosztható build-összetevők. Mielőtt ez a beépülő modul használható lenne a Jenkins-feladatokkal, konfigurálni kell egy Azure Storage-fiókot.

### <a name="jenkins-azure-plugins"></a>Jenkins Azure beépülő modulok

Az Azure-hoz készült Jenkins megoldássablon több Azure beépülő modult telepít. Az Azure DevOps csapata hozza létre és tartja karban a megoldássablont és az alábbi beépülő modulokat, amelyek az Azure Marketplace áruházban elérhető egyéb Jenkins ajánlatokkal és a helyszínen telepített Jenkins főkiszolgálókkal is használhatók:

-   Az [Azure AD beépülő modul][configure-azure-ad] használatával a Jenkins-kiszolgáló támogatja a felhasználók egyszeri bejelentkeztetését az Azure AD alapján.

-   Az [Azure VM Agents][configure-agent] beépülő modul egy Azure Resource Manager (ARM) sablon használatával Jenkins-ügynököket hoz létre Azure-beli virtuális gépeken.

-   Az [Azure Credentials][configure-credential] beépülő modul segítségével tárolhatók az egyszerű Azure-szolgáltatások a Jenkinsben használt hitelesítő adatai.

-   A [Windows Azure Storage beépülő modul][configure-storage] build-összetevőket tölt fel, vagy build-függőségeket tölt le az [Azure Blob Storage-ból][blob].

Emellett javasoljuk, hogy tekintse át az Azure-erőforrásokkal használható Azure beépülő modulok egyre bővülő listáját. A legfrissebb lista megtekintéséhez keresse fel a [Jenkins beépülő modulok indexét][index], és keressen rá az Azure kifejezésre. Például a következő beépülő modulok érhetőek el az üzembe helyezéshez:

-   Az [Azure Container Agents][container-agents] segítségével a tárolókat ügynökként futtathatja a Jenkinsben.

-   A [Kubernetes Continuous Deploy](https://aka.ms/azjenkinsk8s) erőforrás-konfigurációkat helyez üzembe egy Kubernetes-fürtön.

-   Az [Azure Container Service][acs] konfigurációkat helyez üzembe az Azure Container Service szolgáltatásban a Kubernetes, a DC/OS with Marathon vagy a Docker Swarm használatával.

-   Az [Azure Functions][functions] üzembe helyezi a projektet az Azure Functionsben.

-   Az [Azure App Service][app-service] az Azure App Service-be végzi az üzembe helyezést.

## <a name="scalability-considerations"></a>Méretezési szempontok

A Jenkins felskálázásával rendkívül nagy számítási feladatok is támogathatók. Rugalmas buildek esetében ne a Jenkins-főkiszolgálón futtassa a buildeket. Ehelyett szervezze ki a build-feladatokat Jenkins-ügynökökbe, amelyek aztán igény szerint rugalmasan fel- és leskálázhatóak. Az ügynökök skálázása kapcsán mérlegelje a következő két lehetőséget:

- Az [Azure VM Agents][vm-agent] beépülő modul használatával Azure-beli virtuális gépeken futó Jenkins-ügynököket hozhat létre. A beépülő modul lehetővé teszi az ügynökök rugalmas felskálázását, és különféle típusú virtuális gépeket képes használni. Választhat egy eltérő alaprendszerképet az Azure Marketplace-ről, vagy használhat egyedi rendszerképet is. A Jenkins-ügynökök skálázásával kapcsolatos információkért lásd a [Skálázható architektúrák kialakítása][scale] című szakaszt a Jenkins dokumentációjában.

- Az [Azure Container Agents][container-agents] beépülő modul használatával a tárolókat ügynökként futtathatja az [Azure Container Service with Kubernetes](/azure/container-service/kubernetes/) vagy az [Azure Container Instances](/azure/container-instances/) szolgáltatásban.

A virtuális gépek skálázása általában költségesebb, mint a tárolóké. Ha azonban tárolókat használna a skálázáshoz, a build-folyamatoknak tárolókkal kell futnia.

Emellett az Azure Storage használatával oszthatja meg a folyamat következő szakaszában más build-ügynökök által esetleg használt build-összetevőket.

### <a name="scaling-the-jenkins-server"></a>A Jenkins-kiszolgáló skálázása 

A Jenkins-kiszolgáló virtuális gépét a virtuális gép méretének módosításával skálázhatja. Az [Azure-hoz készült Jenkins megoldássablon][azure-market] alapértelmezés szerint a DS2 v2 méretet (két processzorral, 7 GB) adja meg. Ez a méret egy kisebb vagy közepes csapat számítási feladatait képes kezelni. A virtuális gép méretének módosításához válasszon egy másik alternatívát a kiszolgáló kiépítésekor. 

A megfelelő kiszolgálóméret a számítási feladatok várható méretétől függ. A Jenkins-közösség által fenntartott [választási útmutató][selection-guide] segítségével azonosíthatja a követelményeknek leginkább megfelelő konfigurációt. Az Azure számos [méretet biztosít a Linux rendszerű virtuális gépekhez][sizes-linux], amelyek bármely igényt képesek kiszolgálni. A Jenkins-főkiszolgáló skálázásával kapcsolatos információkért tekintse meg a Jenkins-közösség [bevált gyakorlatait][best-practices], amelyek a Jenkins-főkiszolgáló skálázására is kitérnek.


## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Mérje fel a munkafolyamat a rendelkezésre állásra vonatkozó követelményeit, valamint a Jenkins állapotának helyreállítási lehetőségeit, amennyiben a Jenkin-kiszolgáló leállna. A rendelkezésre állásra vonatkozó követelmények felméréséhez két általános mérőszámot érdemes figyelembe vennie:

-   A Helyreállítási időre vonatkozó célkitűzés (RTO) adja meg, hogy mennyi ideig üzemelhet a rendszer a Jenkins nélkül.

-   A Helyreállítási pont-célkitűzés (RPO) határozza meg az elfogadható adatveszteség mértékét, ha a szolgáltatás leállása érinti a Jenkinst.

A gyakorlatban az RTO és az RPO a redundanciát és a biztonsági mentést jelölik. A rendelkezésre állás nem hardveres helyreállítás kérdése, az ugyanis az Azure részét képezi, hanem a Jenkins-kiszolgáló állapota fenntartásának biztosítását jelenti. Ez a referenciaarchitektúra az [Azure szolgáltatói szerződést][sla] (SLA) alkalmazza, amely 99,9 százalékos üzemidőt garantál az egyes virtuális gépekre vonatkozóan. Amennyiben ez az SLA nem felel meg az üzemidővel kapcsolatos elvárásainak, mindenképp gondoskodjon egy vészhelyreállítási tervről, vagy vegye fontolóra egy [több főkiszolgálós Jenkins-kiszolgáló][multi-master] üzembe helyezését (a jelen dokumentum erre nem tér ki).

Vegye fontolóra vészhelyreállítási [szkriptek][disaster] alkalmazását az üzembe helyezés 7. lépésében, amelyek segítségével egy Azure Storage-fiók hozható létre felügyelt meghajtókkal a Jenkins-kiszolgáló állapotának tárolásához. Ha a Jenkins leáll, visszaállítható az ezen a külön tárfiókon tárolt állapotára.

## <a name="security-considerations"></a>Biztonsági szempontok

Az alábbi eljárások segítenek növelni a Jenkins-kiszolgáló biztonságát, mivel alapállapotában nem biztonságos.

-   Konfiguráljon valamilyen megoldást a Jenkins-kiszolgálóra való biztonságos bejelentkezéshez. A HTTP alapértelmezés szerint nem biztonságos. Ez az architektúra a HTTP protokollt alkalmazza, és egy nyilvános IP-címmel rendelkezik. Vegye fontolóra a [HTTPS beállítását a biztonságos bejelentkezéshez használt Nginx-kiszolgálón][nginx].

    > [!NOTE]
    > Ha SSL-t ad a kiszolgálóhoz, hozzon létre egy NSG-szabályt a Jenkins-alhálózaton a 443-as port megnyitásához. További információért olvassa el [a portok a virtuális gépek számára az Azure Portallal való megnyitását][port443] ismertető cikket.
    > 

-   Gondoskodjon róla, hogy a Jenkins konfigurációja meggátolja a webhelyközi kérések hamisítást (A Jenkins felügyelete \> A globális biztonság konfigurálása). Ez a Microsoft Jenkins Server alapértelmezett beállítása.

-   Konfiguráljon csak olvasási hozzáférést a Jenkins-irányítópulthoz a [Matrix Authorization Strategy beépülő modul][matrix] használatával.

-   Telepítse az [Azure Credentials][configure-credential] beépülő modult az Azure-objektumok, a folyamatban érintett ügynökök és a külső felek által készített összetevők titkainak a Key Vaulttal való kezeléséhez.

-   Hozzon létre egy biztonság profilt, amely meghatározza a felhasználók, szolgáltatások és folyamatügynökök által a feladataik elvégzéséhez szükséges erőforrásokat – többet azonban nem. Ez a lépés kritikus fontosságú lesz a biztonsági beállítások mérlegelése során.

A Jenkins-feladatoknak sokszor van szüksége titkokra a hitelesítést igénylő Azure-szolgáltatások, például az Azure Container Service eléréséhez. Használja a [Key Vaultot][key-vault] az [Azure Credential beépülő modullal][configure-credential] ezeknek a titkoknak a biztonságos kezeléséhez. Használja a Key Vaultot az egyszerű szolgáltatások hitelesítő adatainak, jelszavainak, tokenjeinek és egyéb titkainak tárolásához.

Az Azure-erőforrások biztonsági állapotának egy központi felületen való nyomon követéséhez használja az [Azure Security Centert][security-center]. A Security Center monitorozza a potenciális biztonsági problémákat, és átfogó képet nyújt az üzemi környezet biztonsági állapotáról. A Security Center Azure-előfizetésenként külön konfigurálandó. Engedélyezze a biztonsági adatok gyűjtését [az Azure Security Center rövid útmutatójában leírtak szerint][quick-start]. Az adatgyűjtés engedélyezése esetén a Security Center automatikusan figyeli az előfizetés alatt létrehozott összes virtuális gépet.

A Jenkins-kiszolgáló saját felhasználókezelő rendszerrel rendelkezik, a Jenkins-közösség pedig bevált gyakorlatokat biztosít [a Jenkins-példányok biztosításához az Azure-ban][secure-jenkins]. Az Azure-hoz készült Jenkins megoldássablonba ezek a bevált gyakorlatok be vannak építve.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Használjon erőforráscsoportokat az üzembe helyezett Azure-erőforrások rendezéséhez. Az éles és a fejlesztési/tesztkörnyezeteket külön erőforráscsoportokban helyezze üzembe, így az egyes környezetek erőforrásait külön monitorozhatja, és a költségeket erőforráscsoportonként követheti. Az erőforrásokat törölheti is készletenként. Ez nagyon hasznos tesztkörnyezetek esetében.

Az Azure számos különféle szolgáltatást nyújt az infrastruktúra átfogó [monitorozásához és diagnosztizálásához][monitoring-diag]. A processzorhasználat monitorozásához az architektúra telepíti az Azure Monitort. Az Azure Monitor használatával például monitorozhatja a processzorkihasználtságot, és értesítéseket kaphat, ha a kihasználtság meghaladja a 80 százalékot. (Magas processzorkihasználtság esetén érdemes fontolóra vennie a Jenkins-kiszolgálót futtató virtuális gép felskálázását.) Továbbá értesítést küldhet egy meghatározott felhasználónak, ha a virtuális gép leáll vagy elérhetetlenné válik.

## <a name="communities"></a>Közösségek

A közösségek választ adhatnak a kérdéseire, továbbá segíthetnek a sikeres üzembe helyezésben. A következőket ajánljuk figyelmébe:

-   [A Jenkins-közösség blogja](https://jenkins.io/node/)
-   [Az Azure fórum](https://azure.microsoft.com/support/forums/)
-   [Stack Overflow Jenkins](https://stackoverflow.com/tags/jenkins/info)

A Jenkins-közösség további bevált gyakorlataiért látogasson el a [Jenkins bevált gyakorlatait ismertető oldalra][jenkins-best].

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Az architektúra üzembe helyezéséhez kövesse [az Azure-hoz készült Jenkins megoldássablon][azure-market] telepítésének lépéseit, majd telepítse a szkripteket, amelyek az alábbi lépésekben a monitorozás és a vészhelyreállítás beállítását végzik.

### <a name="prerequisites"></a>Előfeltételek

- A referenciaarchitektúra használatához Azure-előfizetés szükséges. 
- Egyszerű Azure-szolgáltatások létrehozásához rendszergazdai jogosultság szükséges az üzembe helyezett Jenkins-kiszolgálóval társított Azure AD-bérlőn.
- A jelen utasítások feltételezik, hogy a Jenkins-rendszergazda egyben Azure-felhasználó is, aki legalább Közreműködői jogosultságokkal rendelkezik.

### <a name="step-1-deploy-the-jenkins-server"></a>1. lépés: A Jenkins-kiszolgáló üzembe helyezése

1.  Nyissa meg a [Jenkins Azure Marketplace-beli rendszerképét][azure-market] a webböngészőben, és az oldal bal oldalán válassza a **LETÖLTÉS MOST** lehetőséget.

2.  Tekintse át a díjszabás részleteit, és válassza a **Folytatás**, majd a **Létrehozás** lehetőséget a Jenkins-kiszolgáló Azure Portalon történő konfigurálásához.

Részletes utasításokért lásd [a Jenkins-kiszolgáló Azure-beli linuxos virtuális gépen az Azure Portalról való létrehozását][create-jenkins] ismertető szakaszt. Ehhez a referenciaarchitektúrához elegendő a kiszolgálót rendszergazdai bejelentkezéssel telepíteni és beüzemelni. Ezután konfigurálhatja, hogy más szolgáltatásokat is futtasson.

### <a name="step-2-set-up-sso"></a>2. lépés: Az egyszeri bejelentkezés beállítása

Ezt a lépést a Jenkins-rendszergazda futtatja, akinek rendelkeznie kell egy felhasználói fiókkal az előfizetés Azure AD könyvtárában, továbbá a Közreműködői szerepkörrel is.

Használja az [Azure AD beépülő modult][configure-azure-ad] a Jenkins frissítési központjából a Jenkins-kiszolgálón, és kövesse az utasításokat az egyszeri bejelentkezés konfigurálásához.

### <a name="step-3-provision-jenkins-server-with-azure-vm-agent-plugin"></a>3. lépés: A Jenkins-kiszolgáló konfigurálása az Azure VM Agent beépülő modullal

Ezt a lépést a Jenkins-rendszergazda futtatja a már telepített Azure VM Agent beépülő modul konfigurálása érdekében.

[A beépülő modul konfigurálásához kövesse ezeket a lépéseket][configure-agent]. A beépülő modulhoz telepítendő egyszerű szolgáltatásokkal kapcsolatos oktatóanyagért olvassa át [a Jenkins üzemi környezetek az igényeknek való megfelelés érdekében az Azure VM-ügynökökkel történő skálázását][scale-agent] ismertető szakaszt.

### <a name="step-4-provision-jenkins-server-with-azure-storage"></a>4. lépés: A Jenkins-kiszolgáló konfigurálása az Azure Storage szolgáltatással

Ezt a lépést a Jenkins-rendszergazda futtatja a már telepített Windows Azure Storage beépülő modul konfigurálása érdekében.

[A beépülő modul konfigurálásához kövesse ezeket a lépéseket][configure-storage].

### <a name="step-5-provision-jenkins-server-with-azure-credential-plugin"></a>5. lépés: A Jenkins-kiszolgáló konfigurálása az Azure Credential beépülő modullal

Ezt a lépést a Jenkins-rendszergazda futtatja a már telepített Azure Credential beépülő modul konfigurálása érdekében.

[A beépülő modul konfigurálásához kövesse ezeket a lépéseket][configure-credential].

### <a name="step-6-provision-jenkins-server-for-monitoring-by-the-azure-monitor-service"></a>6. lépés: A Jenkins-kiszolgáló konfigurálása az Azure Monitor Service általi monitoroztatásra

A Jenkins-kiszolgáló monitorozásának konfigurálásához kövesse [az Azure-szolgáltatások metrikus riasztásainak az Azure Monitor szolgáltatásban való létrehozását][create-metric] ismertető szakaszt.

### <a name="step-7-provision-jenkins-server-with-managed-disks-for-disaster-recovery"></a>7. lépés: A Jenkins-kiszolgáló konfigurálása a Managed Disks szolgáltatással vészhelyreállítás céljából

A Microsoft Jenkins terméket fejlesztő csapat által létrehozott vészhelyreállítási szkriptek felügyelt lemezeket hoznak létre a Jenkins állapotának mentéséhez. Ha a kiszolgáló leáll, visszaállítható a legutóbbi állapotára.

Töltse le és futtassa a vészhelyreállítási szkripteket a [GitHubról][disaster].

[acs]: https://aka.ms/azjenkinsacs
[ad-sp]: /azure/active-directory/develop/active-directory-integrating-applications
[app-service]: https://plugins.jenkins.io/azure-app-service
[azure-ad]: /azure/active-directory/
[azure-market]: https://azuremarketplace.microsoft.com/marketplace/apps/azure-oss.jenkins?tab=Overview
[best-practices]: https://jenkins.io/doc/book/architecting-for-scale/
[blob]: /azure/storage/common/storage-java-jenkins-continuous-integration-solution
[configure-azure-ad]: https://plugins.jenkins.io/azure-ad
[configure-agent]: https://plugins.jenkins.io/azure-vm-agents
[configure-credential]: https://plugins.jenkins.io/azure-credentials
[configure-storage]: https://plugins.jenkins.io/windows-azure-storage
[container-agents]: https://aka.ms/azcontaineragent
[create-jenkins]: /azure/jenkins/install-jenkins-solution-template
[create-metric]: /azure/monitoring-and-diagnostics/insights-alerts-portal
[disaster]: https://github.com/Azure/jenkins/tree/master/disaster_recovery
[functions]: https://aka.ms/azjenkinsfunctions
[index]: https://plugins.jenkins.io
[jenkins-best]: https://wiki.jenkins.io/display/JENKINS/Jenkins+Best+Practices
[jenkins-on-azure]: /azure/jenkins/
[key-vault]: /azure/key-vault/
[managed-disk]: /azure/virtual-machines/linux/managed-disks-overview
[matrix]: https://plugins.jenkins.io/matrix-auth
[monitor]: /azure/monitoring-and-diagnostics/
[monitoring-diag]: /azure/architecture/best-practices/monitoring
[multi-master]: https://jenkins.io/doc/book/architecting-for-scale/
[nginx]: https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04
[nsg]: /azure/virtual-network/virtual-networks-nsg
[quick-start]: /azure/security-center/security-center-get-started
[port443]: /azure/virtual-machines/windows/nsg-quickstart-portal
[premium]: /azure/virtual-machines/linux/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rg]: /azure/azure-resource-manager/resource-group-overview
[scale]: https://jenkins.io/doc/book/architecting-for-scale/
[scale-agent]: /azure/jenkins/jenkins-azure-vm-agents
[selection-guide]: https://jenkins.io/doc/book/hardware-recommendations/
[service-principal]: /azure/active-directory/develop/active-directory-application-objects
[secure-jenkins]: https://jenkins.io/blog/2017/04/20/secure-jenkins-on-azure/
[security-center]: /azure/security-center/security-center-intro
[sizes-linux]: /azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[solution]: https://azure.microsoft.com/blog/announcing-the-solution-template-for-jenkins-on-azure/
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
[storage-plugin]: https://wiki.jenkins.io/display/JENKINS/Windows+Azure+Storage+Plugin
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[vm-agent]: https://wiki.jenkins.io/display/JENKINS/Azure+VM+Agents+plugin
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/jenkins-server.png 
