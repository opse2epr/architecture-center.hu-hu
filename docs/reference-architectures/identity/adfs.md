---
title: Kiterjesztheti a helyszíni AD FS az Azure-bA
titleSuffix: Azure Reference Architectures
description: Az Active Directory összevonási szolgáltatás engedélyezésével biztonságos hibrid hálózati architektúra megvalósítása az Azure-ban.
author: telmosampaio
ms.date: 12/18.2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, identity
ms.openlocfilehash: 873b6a86da14e00d0a537f910d10922444cc1ded
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640736"
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a><span data-ttu-id="43d1a-103">Az Active Directory összevonási szolgáltatások (AD FS) kiterjesztése az Azure-ra</span><span class="sxs-lookup"><span data-stu-id="43d1a-103">Extend Active Directory Federation Services (AD FS) to Azure</span></span>

<span data-ttu-id="43d1a-104">Ez a referenciaarchitektúra egy olyan biztonságos hibrid hálózatot valósít meg, amely kiterjeszti a helyszíni hálózatot az Azure-ra, és az [Active Directory összevonási szolgáltatások (AD FS)][active-directory-federation-services] használatával összevont hitelesítést és engedélyezést végez az Azure-ban futó összetevők számára.</span><span class="sxs-lookup"><span data-stu-id="43d1a-104">This reference architecture implements a secure hybrid network that extends your on-premises network to Azure and uses [Active Directory Federation Services (AD FS)][active-directory-federation-services] to perform federated authentication and authorization for components running in Azure.</span></span> <span data-ttu-id="43d1a-105">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="43d1a-105">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Biztonságos hibrid hálózati architektúra az Active Directoryval](./images/adfs.png)

<span data-ttu-id="43d1a-107">*Töltse le az architektúra [Visio-fájlját][visio-download].*</span><span class="sxs-lookup"><span data-stu-id="43d1a-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="43d1a-108">Az AD FS üzemeltethető a helyszínen, de ha az alkalmazás egy hibrid, amelynek egyes részei az Azure-ban futnak, az AD FS replikálása a felhőben hatékonyabb megoldás lehet.</span><span class="sxs-lookup"><span data-stu-id="43d1a-108">AD FS can be hosted on-premises, but if your application is a hybrid in which some parts are implemented in Azure, it may be more efficient to replicate AD FS in the cloud.</span></span>

<span data-ttu-id="43d1a-109">A diagram a következő eseteket mutatja be:</span><span class="sxs-lookup"><span data-stu-id="43d1a-109">The diagram shows the following scenarios:</span></span>

- <span data-ttu-id="43d1a-110">Egy partnerszervezettől származó alkalmazáskód hozzáfér egy, az Ön Azure-beli virtuális hálózatán futó webalkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="43d1a-110">Application code from a partner organization accesses a web application hosted inside your Azure VNet.</span></span>
- <span data-ttu-id="43d1a-111">Egy külső, az Active Directory Domain Servicesben (DS) tárolt hitelesítő adatokkal rendelkező regisztrált felhasználó hozzáfér az Ön Azure-beli virtuális hálózatán futó webalkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="43d1a-111">An external, registered user with credentials stored inside Active Directory Domain Services (DS) accesses a web application hosted inside your Azure VNet.</span></span>
- <span data-ttu-id="43d1a-112">Egy felhasználó, aki hitelesített eszközről csatlakozik az Ön virtuális hálózatához, elindít egy webalkalmazást, amely az Ön Azure virtuális hálózatán belül fut.</span><span class="sxs-lookup"><span data-stu-id="43d1a-112">A user connected to your VNet using an authorized device executes a web application hosted inside your Azure VNet.</span></span>

<span data-ttu-id="43d1a-113">Az architektúra gyakori használati módjai többek között a következők:</span><span class="sxs-lookup"><span data-stu-id="43d1a-113">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="43d1a-114">Hibrid alkalmazások, amelyekben a számítási feladatok részben a helyszínen, részben pedig az Azure-ban futnak.</span><span class="sxs-lookup"><span data-stu-id="43d1a-114">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
- <span data-ttu-id="43d1a-115">Megoldások, amelyek összevont engedélyezéssel tesznek elérhetővé webalkalmazásokat partnerszervezetek számára.</span><span class="sxs-lookup"><span data-stu-id="43d1a-115">Solutions that use federated authorization to expose web applications to partner organizations.</span></span>
- <span data-ttu-id="43d1a-116">Rendszerek, amelyek támogatják a hozzáférést a vállalati tűzfalon kívül futó webböngészőknek.</span><span class="sxs-lookup"><span data-stu-id="43d1a-116">Systems that support access from web browsers running outside of the organizational firewall.</span></span>
- <span data-ttu-id="43d1a-117">Rendszerek, amelyek lehetővé teszik a felhasználók számára a webes alkalmazásokhoz való hozzáférést hitelesített külső eszközökkel, például távoli számítógépekkel, noteszgépekkel és egyéb mobileszközökkel.</span><span class="sxs-lookup"><span data-stu-id="43d1a-117">Systems that enable users to access to web applications by connecting from authorized external devices such as remote computers, notebooks, and other mobile devices.</span></span>

<span data-ttu-id="43d1a-118">Ez a referenciaarchitektúra a *passzív összevonásra* összpontosít, amelyben az összevonási kiszolgálók döntenek arról, hogy hogyan és mikor hitelesítsenek egy adott felhasználót.</span><span class="sxs-lookup"><span data-stu-id="43d1a-118">This reference architecture focuses on *passive federation*, in which the federation servers decide how and when to authenticate a user.</span></span> <span data-ttu-id="43d1a-119">A felhasználó az alkalmazás indításakor megadja a bejelentkezési adatokat.</span><span class="sxs-lookup"><span data-stu-id="43d1a-119">The user provides sign in information when the application is started.</span></span> <span data-ttu-id="43d1a-120">Ezt a mechanizmust legtöbbször webböngészők használják. Egy olyan protokollt alkalmaz, amely átirányítja a felhasználót arra az oldalra, ahol elvégezheti a hitelesítést.</span><span class="sxs-lookup"><span data-stu-id="43d1a-120">This mechanism is most commonly used by web browsers and involves a protocol that redirects the browser to a site where the user authenticates.</span></span> <span data-ttu-id="43d1a-121">Az AD FS az *aktív összevonást* is támogatja. Ennek során egy alkalmazás felelőssége a hitelesítő adatok biztosítása, további felhasználói interakció igénylése nélkül – erre az esetre azonban ez az architektúra nem terjed ki.</span><span class="sxs-lookup"><span data-stu-id="43d1a-121">AD FS also supports *active federation*, where an application takes on responsibility for supplying credentials without further user interaction, but that scenario is outside the scope of this architecture.</span></span>

<span data-ttu-id="43d1a-122">További szempontok: [Megoldás választása a helyszíni Active Directory Azure-ral való integrálásához][considerations].</span><span class="sxs-lookup"><span data-stu-id="43d1a-122">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span>

## <a name="architecture"></a><span data-ttu-id="43d1a-123">Architektúra</span><span class="sxs-lookup"><span data-stu-id="43d1a-123">Architecture</span></span>

<span data-ttu-id="43d1a-124">Ez az architektúra kiterjeszti [Az AD DS kiterjesztése az Azure-ra][extending-ad-to-azure] című részben leírt megvalósítást.</span><span class="sxs-lookup"><span data-stu-id="43d1a-124">This architecture extends the implementation described in [Extending AD DS to Azure][extending-ad-to-azure].</span></span> <span data-ttu-id="43d1a-125">Az alábbi összetevőket tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="43d1a-125">It contains the followign components.</span></span>

- <span data-ttu-id="43d1a-126">**AD DS-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="43d1a-126">**AD DS subnet**.</span></span> <span data-ttu-id="43d1a-127">Az AD DS-kiszolgálók a saját alhálózatukban találhatók. Tűzfalként a hálózati biztonsági csoport (NSG) szabályai szolgálnak.</span><span class="sxs-lookup"><span data-stu-id="43d1a-127">The AD DS servers are contained in their own subnet with network security group (NSG) rules acting as a firewall.</span></span>

- <span data-ttu-id="43d1a-128">**AD DS-kiszolgálók**.</span><span class="sxs-lookup"><span data-stu-id="43d1a-128">**AD DS servers**.</span></span> <span data-ttu-id="43d1a-129">Az Azure-ban virtuális gépekként futó tartományvezérlők.</span><span class="sxs-lookup"><span data-stu-id="43d1a-129">Domain controllers running as VMs in Azure.</span></span> <span data-ttu-id="43d1a-130">Ezek a kiszolgálók végzik a helyi identitások hitelesítését a tartományon belül.</span><span class="sxs-lookup"><span data-stu-id="43d1a-130">These servers provide authentication of local identities within the domain.</span></span>

- <span data-ttu-id="43d1a-131">**AD FS-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="43d1a-131">**AD FS subnet**.</span></span> <span data-ttu-id="43d1a-132">Az AD FS-kiszolgálók a saját alhálózatukban találhatók. Tűzfalként az NSG-szabályok szolgálnak.</span><span class="sxs-lookup"><span data-stu-id="43d1a-132">The AD FS servers are located within their own subnet with NSG rules acting as a firewall.</span></span>

- <span data-ttu-id="43d1a-133">**AD FS-kiszolgálók**.</span><span class="sxs-lookup"><span data-stu-id="43d1a-133">**AD FS servers**.</span></span> <span data-ttu-id="43d1a-134">Az AD FS-kiszolgálók összevont engedélyezést és hitelesítést biztosítanak.</span><span class="sxs-lookup"><span data-stu-id="43d1a-134">The AD FS servers provide federated authorization and authentication.</span></span> <span data-ttu-id="43d1a-135">Ebben az architektúrában a következő feladatokat látják el:</span><span class="sxs-lookup"><span data-stu-id="43d1a-135">In this architecture, they perform the following tasks:</span></span>

  - <span data-ttu-id="43d1a-136">A partner összevonási kiszolgáló által partnerfelhasználó nevében küldött jogcímeket tartalmazó biztonsági jogkivonatok fogadása.</span><span class="sxs-lookup"><span data-stu-id="43d1a-136">Receiving security tokens containing claims made by a partner federation server on behalf of a partner user.</span></span> <span data-ttu-id="43d1a-137">Az AD FS ellenőrzi, hogy a jogkivonatok érvényesek-e, mielőtt átadja azokat az Azure-ban futó webalkalmazásnak, amely a kérelmek engedélyezését végzi.</span><span class="sxs-lookup"><span data-stu-id="43d1a-137">AD FS verifies that the tokens are valid before passing the claims to the web application running in Azure to authorize requests.</span></span>

    <span data-ttu-id="43d1a-138">Az Azure-ban futó alkalmazások a *függő*.</span><span class="sxs-lookup"><span data-stu-id="43d1a-138">The application running in Azure is the *relying party*.</span></span> <span data-ttu-id="43d1a-139">A partner összevonási kiszolgálónak olyan jogcímeket kell kiadni, amelyeket a webalkalmazás értelmezni tud.</span><span class="sxs-lookup"><span data-stu-id="43d1a-139">The partner federation server must issue claims that are understood by the web application.</span></span> <span data-ttu-id="43d1a-140">A partner összevonási kiszolgálókat *fiókpartnereknek* nevezik, mert ezek küldenek hozzáférési kérelmeket a partnerszervezet hitelesített fiókjainak nevében.</span><span class="sxs-lookup"><span data-stu-id="43d1a-140">The partner federation servers are referred to as *account partners*, because they submit access requests on behalf of authenticated accounts in the partner organization.</span></span> <span data-ttu-id="43d1a-141">Az AD FS-kiszolgálókat *erőforráspartnereknek* nevezik, mert ezek biztosítanak hozzáférést az erőforrásokhoz (a webalkalmazáshoz).</span><span class="sxs-lookup"><span data-stu-id="43d1a-141">The AD FS servers are called *resource partners* because they provide access to resources (the web application).</span></span>

  - <span data-ttu-id="43d1a-142">Webböngészőt vagy webalkalmazás-hozzáférést igénylő eszközt futtató külső felhasználóktól érkező kérelmek hitelesítése és engedélyezése az AD DS-sel és az [Active Directory eszközregisztrációs szolgáltatásával][ADDRS].</span><span class="sxs-lookup"><span data-stu-id="43d1a-142">Authenticating and authorizing incoming requests from external users running a web browser or device that needs access to web applications, by using AD DS and the [Active Directory Device Registration Service][ADDRS].</span></span>

  <span data-ttu-id="43d1a-143">Az AD FS-kiszolgálók farmként vannak konfigurálva, amely egy Azure-terheléselosztón keresztül érhető el.</span><span class="sxs-lookup"><span data-stu-id="43d1a-143">The AD FS servers are configured as a farm accessed through an Azure load balancer.</span></span> <span data-ttu-id="43d1a-144">Ez a megvalósítás javítja a rendelkezésre állást és a méretezhetőséget.</span><span class="sxs-lookup"><span data-stu-id="43d1a-144">This implementation improves availability and scalability.</span></span> <span data-ttu-id="43d1a-145">Az AD FS-kiszolgálók nem érhetők el közvetlenül az internetről.</span><span class="sxs-lookup"><span data-stu-id="43d1a-145">The AD FS servers are not exposed directly to the Internet.</span></span> <span data-ttu-id="43d1a-146">Minden internetes forgalom szűrve van AD FS webalkalmazás-proxy kiszolgálókon és egy szegélyhálózaton (DMZ) keresztül.</span><span class="sxs-lookup"><span data-stu-id="43d1a-146">All Internet traffic is filtered through AD FS web application proxy servers and a DMZ (also referred to as a perimeter network).</span></span>

  <span data-ttu-id="43d1a-147">Információk az AD FS működéséről: [Active Directory összevonási szolgáltatások – áttekintés][active-directory-federation-services-overview].</span><span class="sxs-lookup"><span data-stu-id="43d1a-147">For more information about how AD FS works, see [Active Directory Federation Services Overview][active-directory-federation-services-overview].</span></span> <span data-ttu-id="43d1a-148">Emellett [Az AD FS üzembe helyezése az Azure-ban][adfs-intro] című cikk lépésről lépésre bemutatja a megvalósítást.</span><span class="sxs-lookup"><span data-stu-id="43d1a-148">Also, the article [AD FS deployment in Azure][adfs-intro] contains a detailed step-by-step introduction to implementation.</span></span>

- <span data-ttu-id="43d1a-149">**AD FS proxyalhálózat**.</span><span class="sxs-lookup"><span data-stu-id="43d1a-149">**AD FS proxy subnet**.</span></span> <span data-ttu-id="43d1a-150">Az AD FS-proxykiszolgálók lehetnek a saját alhálózatukon belül, ahol védelmüket az NSG-szabályok biztosítják.</span><span class="sxs-lookup"><span data-stu-id="43d1a-150">The AD FS proxy servers can be contained within their own subnet, with NSG rules providing protection.</span></span> <span data-ttu-id="43d1a-151">Az alhálózaton belüli kiszolgálók hálózati virtuális készülékeken keresztül kapcsolódnak az internethez. Ezek tűzfalat biztosítanak az Azure virtuális hálózat és az internet között.</span><span class="sxs-lookup"><span data-stu-id="43d1a-151">The servers in this subnet are exposed to the Internet through a set of network virtual appliances that provide a firewall between your Azure virtual network and the Internet.</span></span>

- <span data-ttu-id="43d1a-152">**AD FS webalkalmazás-proxy (WAP) kiszolgálók**.</span><span class="sxs-lookup"><span data-stu-id="43d1a-152">**AD FS web application proxy (WAP) servers**.</span></span> <span data-ttu-id="43d1a-153">Ezek a virtuális gépek AD FS-kiszolgálókként működnek a partnerszervezetektől és a külső eszközöktől érkező kérelmek számára.</span><span class="sxs-lookup"><span data-stu-id="43d1a-153">These VMs act as AD FS servers for incoming requests from partner organizations and external devices.</span></span> <span data-ttu-id="43d1a-154">A WAP-kiszolgálók szűrőként működnek, védelmet képezve az AD FS-kiszolgálók számára az internetről való közvetlen hozzáféréssel szemben.</span><span class="sxs-lookup"><span data-stu-id="43d1a-154">The WAP servers act as a filter, shielding the AD FS servers from direct access from the Internet.</span></span> <span data-ttu-id="43d1a-155">Ahogy az AD FS-kiszolgálók esetében is, a WAP-kiszolgálók farmban, terheléselosztással való üzemeltetése jobb rendelkezésre állást és méretezhetőséget biztosít, mint ha különálló kiszolgálók gyűjteményeként működtetné azokat.</span><span class="sxs-lookup"><span data-stu-id="43d1a-155">As with the AD FS servers, deploying the WAP servers in a farm with load balancing gives you greater availability and scalability than deploying a collection of stand-alone servers.</span></span>

  > [!NOTE]
  > <span data-ttu-id="43d1a-156">Részletes információk a WAP-kiszolgálók telepítéséről: [A webalkalmazás-proxy kiszolgáló telepítése és konfigurálása][install_and_configure_the_web_application_proxy_server]</span><span class="sxs-lookup"><span data-stu-id="43d1a-156">For detailed information about installing WAP servers, see [Install and Configure the Web Application Proxy Server][install_and_configure_the_web_application_proxy_server]</span></span>
  >

- <span data-ttu-id="43d1a-157">**Partnerszervezet**.</span><span class="sxs-lookup"><span data-stu-id="43d1a-157">**Partner organization**.</span></span> <span data-ttu-id="43d1a-158">Egy Azure-beli webalkalmazáshoz való hozzáférést kérő webes alkalmazást futtató partnerszervezet.</span><span class="sxs-lookup"><span data-stu-id="43d1a-158">A partner organization running a web application that requests access to a web application running in Azure.</span></span> <span data-ttu-id="43d1a-159">A partnerszervezet összevonási kiszolgálója helyileg hitelesíti a kérelmeket, és elküldi a jogcímeket tartalmazó biztonsági jogkivonatokat az Azure-ban futó AD FS-nek.</span><span class="sxs-lookup"><span data-stu-id="43d1a-159">The federation server at the partner organization authenticates requests locally, and submits security tokens containing claims to AD FS running in Azure.</span></span> <span data-ttu-id="43d1a-160">Az AD FS az Azure-ban érvényesíti a biztonsági jogkivonatokat, és az érvényeseket továbbítja az Azure-beli webalkalmazásnak engedélyezés céljából.</span><span class="sxs-lookup"><span data-stu-id="43d1a-160">AD FS in Azure validates the security tokens, and if valid can pass the claims to the web application running in Azure to authorize them.</span></span>

  > [!NOTE]
  > <span data-ttu-id="43d1a-161">Emellett konfigurálhat egy VPN-alagutat is az Azure-átjáróval, amelyen keresztül közvetlen hozzáférést biztosíthat a megbízható partnerek számára.</span><span class="sxs-lookup"><span data-stu-id="43d1a-161">You can also configure a VPN tunnel using Azure gateway to provide direct access to AD FS for trusted partners.</span></span> <span data-ttu-id="43d1a-162">Az ezen partnerektől érkező kérelmek nem haladnak át a WAP-kiszolgálókon.</span><span class="sxs-lookup"><span data-stu-id="43d1a-162">Requests received from these partners do not pass through the WAP servers.</span></span>
  >

## <a name="recommendations"></a><span data-ttu-id="43d1a-163">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="43d1a-163">Recommendations</span></span>

<span data-ttu-id="43d1a-164">Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="43d1a-164">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="43d1a-165">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="43d1a-165">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="networking-recommendations"></a><span data-ttu-id="43d1a-166">Hálózatokra vonatkozó javaslatok</span><span class="sxs-lookup"><span data-stu-id="43d1a-166">Networking recommendations</span></span>

<span data-ttu-id="43d1a-167">A hálózati adaptert minden, AD FS- és WAP-kiszolgálót futtató virtuális géphez statikus magánhálózati IP-címmel konfigurálja.</span><span class="sxs-lookup"><span data-stu-id="43d1a-167">Configure the network interface for each of the VMs hosting AD FS and WAP servers with static private IP addresses.</span></span>

<span data-ttu-id="43d1a-168">Ne adjon nyilvános IP-címeket az AD FS-t futtató virtuális gépeknek.</span><span class="sxs-lookup"><span data-stu-id="43d1a-168">Do not give the AD FS VMs public IP addresses.</span></span> <span data-ttu-id="43d1a-169">További információkért lásd: a [biztonsági szempontok](#security-considerations) szakaszban.</span><span class="sxs-lookup"><span data-stu-id="43d1a-169">For more information, see the [Security considerations](#security-considerations) section.</span></span>

<span data-ttu-id="43d1a-170">Állítsa be az elsődleges és a másodlagos tartománynév-kiszolgálók (DNS) IP-címeit az egyes AD FS és WAP virtuális gépek hálózati adaptereihez úgy, hogy az Active Directory DS virtuális gépeire hivatkozzanak.</span><span class="sxs-lookup"><span data-stu-id="43d1a-170">Set the IP address of the preferred and secondary domain name service (DNS) servers for the network interfaces for each AD FS and WAP VM to reference the Active Directory DS VMs.</span></span> <span data-ttu-id="43d1a-171">Az Active Directory DS virtuális gépeknek DNS kell futtatnia.</span><span class="sxs-lookup"><span data-stu-id="43d1a-171">The Active Directory DS VMs should be running DNS.</span></span> <span data-ttu-id="43d1a-172">Ez a lépés szükséges ahhoz, hogy az egyes virtuális gépek csatlakozhassanak a tartományhoz.</span><span class="sxs-lookup"><span data-stu-id="43d1a-172">This step is necessary to enable each VM to join the domain.</span></span>

### <a name="ad-fs-installation"></a><span data-ttu-id="43d1a-173">Az AD FS telepítése</span><span class="sxs-lookup"><span data-stu-id="43d1a-173">AD FS installation</span></span>

<span data-ttu-id="43d1a-174">Az [Összevonási kiszolgálók farmjának központi telepítése][Deploying_a_federation_server_farm] című cikkben részletes útmutatás található az AD FS telepítéséhez és konfigurálásához.</span><span class="sxs-lookup"><span data-stu-id="43d1a-174">The article [Deploying a Federation Server Farm][Deploying_a_federation_server_farm] provides detailed instructions for installing and configuring AD FS.</span></span> <span data-ttu-id="43d1a-175">A farm első AD FS-kiszolgálójának konfigurálása előtt végezze el a következő műveleteket:</span><span class="sxs-lookup"><span data-stu-id="43d1a-175">Perform the following tasks before configuring the first AD FS server in the farm:</span></span>

1. <span data-ttu-id="43d1a-176">Szerezzen be egy nyilvánosan megbízható tanúsítványt a kiszolgálók hitelesítéséhez.</span><span class="sxs-lookup"><span data-stu-id="43d1a-176">Obtain a publicly trusted certificate for performing server authentication.</span></span> <span data-ttu-id="43d1a-177">A *tulajdonosnévnek* azt a nevet kell tartalmaznia, amelyet az ügyfelek az összevonási szolgáltatás eléréséhez használnak.</span><span class="sxs-lookup"><span data-stu-id="43d1a-177">The *subject name* must contain the name clients use to access the federation service.</span></span> <span data-ttu-id="43d1a-178">Ez lehet a terheléselosztóhoz regisztrált DNS-név, például *adfs.contoso.com* (biztonsági okokból ne használjon helyettesítő karaktereket tartalmazó neveket, mint például a \**.contoso.com*).</span><span class="sxs-lookup"><span data-stu-id="43d1a-178">This can be the DNS name registered for the load balancer, for example, *adfs.contoso.com* (avoid using wildcard names such as \**.contoso.com*, for security reasons).</span></span> <span data-ttu-id="43d1a-179">Használja ugyanazt a tanúsítványt az összes AD FS-kiszolgáló virtuális gépein.</span><span class="sxs-lookup"><span data-stu-id="43d1a-179">Use the same certificate on all AD FS server VMs.</span></span> <span data-ttu-id="43d1a-180">Tanúsítványt megbízható hitelesítésszolgáltatótól vásárolhat, de ha a vállalata az Active Directory tanúsítványszolgáltatást használja, létrehozhatja a sajátját is.</span><span class="sxs-lookup"><span data-stu-id="43d1a-180">You can purchase a certificate from a trusted certification authority, but if your organization uses Active Directory Certificate Services you can create your own.</span></span>

    <span data-ttu-id="43d1a-181">A *tulajdonos alternatív nevét* az eszközregisztrációs szolgáltatás (DRS) használja a külső eszközökről való hozzáférés engedélyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="43d1a-181">The *subject alternative name* is used by the device registration service (DRS) to enable access from external devices.</span></span> <span data-ttu-id="43d1a-182">Ennek a következő formátumban kell lennie: *enterpriseregistration.contoso.com*.</span><span class="sxs-lookup"><span data-stu-id="43d1a-182">This should be of the form *enterpriseregistration.contoso.com*.</span></span>

    <span data-ttu-id="43d1a-183">További információkért lásd: [Secure Sockets Layer (SSL-) tanúsítvány beszerzése és konfigurálása][adfs_certificates].</span><span class="sxs-lookup"><span data-stu-id="43d1a-183">For more information, see [Obtain and Configure a Secure Sockets Layer (SSL) Certificate for AD FS][adfs_certificates].</span></span>

2. <span data-ttu-id="43d1a-184">A tartományvezérlőn hozzon létre egy új legfelső szintű kulcsot a kulcsszolgáltató szolgáltatás számára.</span><span class="sxs-lookup"><span data-stu-id="43d1a-184">On the domain controller, generate a new root key for the Key Distribution Service.</span></span> <span data-ttu-id="43d1a-185">Az érvényesség kezdetének adja meg az aktuális időpontnál 10 órával korábbi időpontot (ez a konfiguráció csökkenti a késést, amely a kulcsok a tartományban való kiosztásakor és szinkronizálásakor léphet fel).</span><span class="sxs-lookup"><span data-stu-id="43d1a-185">Set the effective time to the current time minus 10 hours (this configuration reduces the delay that can occur in distributing and synchronizing keys across the domain).</span></span> <span data-ttu-id="43d1a-186">Ez a lépés az AD FS-szolgáltatás futtatásához használt csoportos szolgáltatásfiók létrehozásának támogatásához szükséges.</span><span class="sxs-lookup"><span data-stu-id="43d1a-186">This step is necessary to support creating the group service account that is used to run the AD FS service.</span></span> <span data-ttu-id="43d1a-187">A következő PowerShell-parancs erre mutat példát:</span><span class="sxs-lookup"><span data-stu-id="43d1a-187">The following PowerShell command shows an example of how to do this:</span></span>

    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. <span data-ttu-id="43d1a-188">Minden AD FS-kiszolgáló virtuális gépét vegye fel a tartományba.</span><span class="sxs-lookup"><span data-stu-id="43d1a-188">Add each AD FS server VM to the domain.</span></span>

> [!NOTE]
> <span data-ttu-id="43d1a-189">Az AD FS telepítéséhez a tartomány elsődleges tartományvezérlő (PDC) emulátorának mozgó egyedüli főkiszolgálói (FSMO) szerepkörét futtató tartományvezérlőnek futnia kell, és elérhetőnek kell lennie az AD FS virtuális gépekről.</span><span class="sxs-lookup"><span data-stu-id="43d1a-189">To install AD FS, the domain controller running the primary domain controller (PDC) emulator flexible single master operation (FSMO) role for the domain must be running and accessible from the AD FS VMs.</span></span> <span data-ttu-id="43d1a-190"><<RBC: Van mód, hogy ez kevesebb ismétlődő? >></span><span class="sxs-lookup"><span data-stu-id="43d1a-190"><<RBC: Is there a way to make this less repetitive?>></span></span>
>

### <a name="ad-fs-trust"></a><span data-ttu-id="43d1a-191">AD FS-megbízhatóság</span><span class="sxs-lookup"><span data-stu-id="43d1a-191">AD FS trust</span></span>

<span data-ttu-id="43d1a-192">Alakítson ki összevonási megbízhatóságot az üzemelő AD FS és a partnerszervezetek összevonási kiszolgálói között.</span><span class="sxs-lookup"><span data-stu-id="43d1a-192">Establish federation trust between your AD FS installation, and the federation servers of any partner organizations.</span></span> <span data-ttu-id="43d1a-193">Konfigurálja az összes szükséges jogcímszűrést és -leképezést.</span><span class="sxs-lookup"><span data-stu-id="43d1a-193">Configure any claims filtering and mapping required.</span></span>

- <span data-ttu-id="43d1a-194">Minden egyes partnerszervezet fejlesztési és üzemeltetési csapatának meg kell adnia egy függőentitás-megbízhatóságot az AD FS-kiszolgálókon keresztül elérhető webes alkalmazásokhoz.</span><span class="sxs-lookup"><span data-stu-id="43d1a-194">DevOps staff at each partner organization must add a relying party trust for the web applications accessible through your AD FS servers.</span></span>
- <span data-ttu-id="43d1a-195">Vállalata fejlesztési és üzemeltetési csapatának be kell állítania a jogcímszolgáltatói megbízhatóságot ahhoz, hogy lehetővé tegye az AD FS-kiszolgálók számára a partnerszervezetek által biztosított jogcímek megbízhatóként való kezelését.</span><span class="sxs-lookup"><span data-stu-id="43d1a-195">DevOps staff in your organization must configure claims-provider trust to enable your AD FS servers to trust the claims that partner organizations provide.</span></span>
- <span data-ttu-id="43d1a-196">A vállalat fejlesztési és üzemeltetési csapatának emellett azt is be kell állítania, hogy az AD FS átadja a jogcímeket a vállalat szervezetének webalkalmazásainak.</span><span class="sxs-lookup"><span data-stu-id="43d1a-196">DevOps staff in your organization must also configure AD FS to pass claims on to your organization's web applications.</span></span>

<span data-ttu-id="43d1a-197">További információkért lásd: [Összevonási megbízhatóság létrehozása][establishing-federation-trust].</span><span class="sxs-lookup"><span data-stu-id="43d1a-197">For more information, see [Establishing Federation Trust][establishing-federation-trust].</span></span>

<span data-ttu-id="43d1a-198">Tegye közzé vállalata webalkalmazásait, és tegye elérhetővé azokat a külső partnerek számára a WAP-kiszolgálókon keresztüli előhitelesítéssel.</span><span class="sxs-lookup"><span data-stu-id="43d1a-198">Publish your organization's web applications and make them available to external partners by using preauthentication through the WAP servers.</span></span> <span data-ttu-id="43d1a-199">További információkért lásd: [Alkalmazások közzététele AD FS előhitelesítéssel][publish_applications_using_AD_FS_preauthentication]</span><span class="sxs-lookup"><span data-stu-id="43d1a-199">For more information, see [Publish Applications using AD FS Preauthentication][publish_applications_using_AD_FS_preauthentication]</span></span>

<span data-ttu-id="43d1a-200">Az AD FS támogatja a jogkivonatok átalakítását és kiegészítését.</span><span class="sxs-lookup"><span data-stu-id="43d1a-200">AD FS supports token transformation and augmentation.</span></span> <span data-ttu-id="43d1a-201">Az Azure Active Directory nem biztosítja ezt a funkciót.</span><span class="sxs-lookup"><span data-stu-id="43d1a-201">Azure Active Directory does not provide this feature.</span></span> <span data-ttu-id="43d1a-202">Az AD FS-sel a megbízhatósági kapcsolatok beállításakor a következőket teheti:</span><span class="sxs-lookup"><span data-stu-id="43d1a-202">With AD FS, when you set up the trust relationships, you can:</span></span>

- <span data-ttu-id="43d1a-203">Konfigurálhatja a jogcímek átalakítását az engedélyezési szabályokhoz.</span><span class="sxs-lookup"><span data-stu-id="43d1a-203">Configure claim transformations for authorization rules.</span></span> <span data-ttu-id="43d1a-204">Például leképezheti a csoportos biztonságot egy nem Microsoft-partner szervezet úgy, hogy az Active Directory tartományi szolgáltatások hitelesítheti a szervezet által használt ábrázolásból olyan.</span><span class="sxs-lookup"><span data-stu-id="43d1a-204">For example, you can map group security from a representation used by a non-Microsoft partner organization to something that Active Directory DS can authorize in your organization.</span></span>
- <span data-ttu-id="43d1a-205">Átalakíthatja a jogcímeket másik formátumba.</span><span class="sxs-lookup"><span data-stu-id="43d1a-205">Transform claims from one format to another.</span></span> <span data-ttu-id="43d1a-206">Például leképezést végezhet a SAML 2.0-ról a SAML 1.1-re, ha alkalmazása csak a SAML 1.1 formátumú jogcímeket támogatja.</span><span class="sxs-lookup"><span data-stu-id="43d1a-206">For example, you can map from SAML 2.0 to SAML 1.1 if your application only supports SAML 1.1 claims.</span></span>

### <a name="ad-fs-monitoring"></a><span data-ttu-id="43d1a-207">AD FS-figyelés</span><span class="sxs-lookup"><span data-stu-id="43d1a-207">AD FS monitoring</span></span>

<span data-ttu-id="43d1a-208">Az [Active Directory 2012 R2 összevonási szolgáltatásokhoz készült Microsoft System Center felügyeleti csomag][oms-adfs-pack] proaktív és reaktív felügyeletet is biztosít az összevonási kiszolgálóhoz tartozó AD FS üzemelő példányához.</span><span class="sxs-lookup"><span data-stu-id="43d1a-208">The [Microsoft System Center Management Pack for Active Directory Federation Services 2012 R2][oms-adfs-pack] provides both proactive and reactive monitoring of your AD FS deployment for the federation server.</span></span> <span data-ttu-id="43d1a-209">Ez a felügyeleti csomag a következőket figyeli:</span><span class="sxs-lookup"><span data-stu-id="43d1a-209">This management pack monitors:</span></span>

- <span data-ttu-id="43d1a-210">AD FS szolgáltatás által az eseménynaplókban rögzített eseményeket.</span><span class="sxs-lookup"><span data-stu-id="43d1a-210">Events that the AD FS service records in its event logs.</span></span>
- <span data-ttu-id="43d1a-211">Az AD FS teljesítményszámlálói által gyűjtött teljesítményadatokat.</span><span class="sxs-lookup"><span data-stu-id="43d1a-211">The performance data that the AD FS performance counters collect.</span></span>
- <span data-ttu-id="43d1a-212">A AD FS rendszer és a webes alkalmazások (függő entitások) általános állapotát. Emellett riasztásokat biztosít a kritikus fontosságú problémákról és figyelmeztetésekről.</span><span class="sxs-lookup"><span data-stu-id="43d1a-212">The overall health of the AD FS system and web applications (relying parties), and provides alerts for critical issues and warnings.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="43d1a-213">Méretezési szempontok</span><span class="sxs-lookup"><span data-stu-id="43d1a-213">Scalability considerations</span></span>

<span data-ttu-id="43d1a-214">AD FS-farmok méretezésének megkezdéséhez lásd [Az AD FS üzembe helyezésének megtervezése][plan-your-adfs-deployment] című cikkben összefoglalt szempontokat:</span><span class="sxs-lookup"><span data-stu-id="43d1a-214">The following considerations, summarized from the article [Plan your AD FS deployment][plan-your-adfs-deployment], give a starting point for sizing AD FS farms:</span></span>

- <span data-ttu-id="43d1a-215">Ha 1000-nél kevesebb felhasználóval rendelkezik, ne hozzon létre dedikált kiszolgálókat. Ehelyett telepítse az AD FS-t a felhőben található minden egyes Active Directory DS-kiszolgálóra.</span><span class="sxs-lookup"><span data-stu-id="43d1a-215">If you have fewer than 1000 users, do not create dedicated servers, but instead install AD FS on each of the Active Directory DS servers in the cloud.</span></span> <span data-ttu-id="43d1a-216">A rendelkezésre állás fenntartása érdekében győződjön meg arról, hogy legalább két Active Directory DS-kiszolgálóval rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="43d1a-216">Make sure that you have at least two Active Directory DS servers to maintain availability.</span></span> <span data-ttu-id="43d1a-217">Egyetlen WAP-kiszolgálót hozzon létre.</span><span class="sxs-lookup"><span data-stu-id="43d1a-217">Create a single WAP server.</span></span>
- <span data-ttu-id="43d1a-218">Ha felhasználóinak száma 1000 és 15 000 között van, hozzon létre két dedikált AD FS-kiszolgálót és két dedikált WAP-kiszolgálót.</span><span class="sxs-lookup"><span data-stu-id="43d1a-218">If you have between 1000 and 15000 users, create two dedicated AD FS servers and two dedicated WAP servers.</span></span>
- <span data-ttu-id="43d1a-219">Ha felhasználóinak száma 15 000 és 60 000 között van, hozzon létre 3–5 dedikált AD FS-kiszolgálót és legalább két dedikált WAP-kiszolgálót.</span><span class="sxs-lookup"><span data-stu-id="43d1a-219">If you have between 15000 and 60000 users, create between three and five dedicated AD FS servers and at least two dedicated WAP servers.</span></span>

<span data-ttu-id="43d1a-220">Ezeket a szempontok azt feltételezik, hogy kettős négymagos virtuálisgép-méretet (szabványos D4_v2 vagy jobb) használ az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="43d1a-220">These considerations assume that you are using dual quad-core VM (Standard D4_v2, or better) sizes in Azure.</span></span>

<span data-ttu-id="43d1a-221">Ha a belső Windows-adatbázist használja az AD FS konfigurációs adatainak tárolásához, legfeljebb nyolc AD FS-kiszolgálót használhat a farmban.</span><span class="sxs-lookup"><span data-stu-id="43d1a-221">If you are using the Windows Internal Database to store AD FS configuration data, you are limited to eight AD FS servers in the farm.</span></span> <span data-ttu-id="43d1a-222">Ha várhatóan többre lesz szüksége a jövőben, az SQL Servert használja.</span><span class="sxs-lookup"><span data-stu-id="43d1a-222">If you anticipate that you will need more in the future, use SQL Server.</span></span> <span data-ttu-id="43d1a-223">További információkért lásd: [Az AD FS konfigurációs adatbázisának szerepe][adfs-configuration-database].</span><span class="sxs-lookup"><span data-stu-id="43d1a-223">For more information, see [The Role of the AD FS Configuration Database][adfs-configuration-database].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="43d1a-224">Rendelkezésre állási szempontok</span><span class="sxs-lookup"><span data-stu-id="43d1a-224">Availability considerations</span></span>

<span data-ttu-id="43d1a-225">Hozzon létre egy AD FS-farmot legalább két kiszolgálóval a szolgáltatás rendelkezésre állásának növeléséhez.</span><span class="sxs-lookup"><span data-stu-id="43d1a-225">Create an AD FS farm with at least two servers to increase availability of the service.</span></span> <span data-ttu-id="43d1a-226">Használjon különböző tárfiókokat a farm AD FS virtuális gépeihez.</span><span class="sxs-lookup"><span data-stu-id="43d1a-226">Use different storage accounts for each AD FS VM in the farm.</span></span> <span data-ttu-id="43d1a-227">Ez a megközelítés segít biztosítani, hogy egyetlen tárfiók hibája miatt ne váljon elérhetetlenné a teljes farm.</span><span class="sxs-lookup"><span data-stu-id="43d1a-227">This approach helps to ensure that a failure in a single storage account does not make the entire farm inaccessible.</span></span>

<span data-ttu-id="43d1a-228">Hozzon létre külön Azure rendelkezésre állási csoportokat az AD FS és a WAP virtuális gépekhez.</span><span class="sxs-lookup"><span data-stu-id="43d1a-228">Create separate Azure availability sets for the AD FS and WAP VMs.</span></span> <span data-ttu-id="43d1a-229">Ügyeljen arra, hogy legalább két virtuális gép legyen minden egyes csoportban.</span><span class="sxs-lookup"><span data-stu-id="43d1a-229">Ensure that there are at least two VMs in each set.</span></span> <span data-ttu-id="43d1a-230">Az egyes rendelkezésre állási csoportoknak rendelkezniük kell legalább két frissítési tartománnyal és két tartalék tartománnyal.</span><span class="sxs-lookup"><span data-stu-id="43d1a-230">Each availability set must have at least two update domains and two fault domains.</span></span>

<span data-ttu-id="43d1a-231">Az AD FS és a WAP virtuális gépek terheléselosztóit a következőképpen konfigurálja:</span><span class="sxs-lookup"><span data-stu-id="43d1a-231">Configure the load balancers for the AD FS VMs and WAP VMs as follows:</span></span>

- <span data-ttu-id="43d1a-232">Használjon Azure-terheléselosztót a WAP virtuális gépekhez való külső hozzáférés biztosítására, és egy belső terheléselosztót a farm AD FS-kiszolgálóira irányuló terhelés elosztására.</span><span class="sxs-lookup"><span data-stu-id="43d1a-232">Use an Azure load balancer to provide external access to the WAP VMs, and an internal load balancer to distribute the load across the AD FS servers in the farm.</span></span>
- <span data-ttu-id="43d1a-233">Az AD FS- és WAP-kiszolgálók felé kizárólag a 443-as porton (HTTPS) megjelenő forgalmat továbbítsa.</span><span class="sxs-lookup"><span data-stu-id="43d1a-233">Only pass traffic appearing on port 443 (HTTPS) to the AD FS/WAP servers.</span></span>
- <span data-ttu-id="43d1a-234">A terheléselosztónak statikus IP-címet adjon.</span><span class="sxs-lookup"><span data-stu-id="43d1a-234">Give the load balancer a static IP address.</span></span>
- <span data-ttu-id="43d1a-235">Hozzon létre egy állapotmintát elleni HTTP használatával `/adfs/probe`.</span><span class="sxs-lookup"><span data-stu-id="43d1a-235">Create a health probe using HTTP against `/adfs/probe`.</span></span> <span data-ttu-id="43d1a-236">További információkért lásd: [hardver Load Balancer állapot-ellenőrzések és a webalkalmazás-Proxy / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/).</span><span class="sxs-lookup"><span data-stu-id="43d1a-236">For more information, see [Hardware Load Balancer Health Checks and Web Application Proxy / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/).</span></span>

  > [!NOTE]
  > <span data-ttu-id="43d1a-237">Az AD FS-kiszolgálók a Kiszolgálónév jelzése (SNI) protokollt használják, így ha egy HTTPS-végpont használatával próbál mintát venni, a terheléselosztó hibába ütközik.</span><span class="sxs-lookup"><span data-stu-id="43d1a-237">AD FS servers use the Server Name Indication (SNI) protocol, so attempting to probe using an HTTPS endpoint from the load balancer fails.</span></span>
  >

- <span data-ttu-id="43d1a-238">Adja hozzá egy DNS *A*-rekordot az AD FS-terheléselosztó tartományához.</span><span class="sxs-lookup"><span data-stu-id="43d1a-238">Add a DNS *A* record to the domain for the AD FS load balancer.</span></span> <span data-ttu-id="43d1a-239">Adja meg a terheléselosztó IP-címét, és adjon neki nevet a tartományban (például adfs.contoso.com).</span><span class="sxs-lookup"><span data-stu-id="43d1a-239">Specify the IP address of the load balancer, and give it a name in the domain (such as adfs.contoso.com).</span></span> <span data-ttu-id="43d1a-240">Ezt a nevet használják az ügyfelek és a WAP-kiszolgálók az AD FS-kiszolgálófarm eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="43d1a-240">This is the name clients and the WAP servers use to access the AD FS server farm.</span></span>

<span data-ttu-id="43d1a-241">Az AD FS konfigurációs adatainak tárolására használhatja az SQL Servert vagy a belső Windows-adatbázist.</span><span class="sxs-lookup"><span data-stu-id="43d1a-241">You can use either SQL Server or the Windows Internal Database to hold AD FS configuration information.</span></span> <span data-ttu-id="43d1a-242">A belső Windows-adatbázis alapvető redundanciát biztosít.</span><span class="sxs-lookup"><span data-stu-id="43d1a-242">The Windows Internal Database provides basic redundancy.</span></span> <span data-ttu-id="43d1a-243">A rendszer kizárólag az AD FS-fürt egyik AD FS-adatbázisába írja a módosításokat. A többi kiszolgáló leküldéses replikációval tartja naprakészen az adatbázisokat.</span><span class="sxs-lookup"><span data-stu-id="43d1a-243">Changes are written directly to only one of the AD FS databases in the AD FS cluster, while the other servers use pull replication to keep their databases up to date.</span></span> <span data-ttu-id="43d1a-244">Az SQL Server használata teljes adatbázis-redundanciát és magas rendelkezésre állást biztosíthat feladatátvételi fürtszolgáltatással vagy tükrözéssel.</span><span class="sxs-lookup"><span data-stu-id="43d1a-244">Using SQL Server can provide full database redundancy and high availability using failover clustering or mirroring.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="43d1a-245">Felügyeleti szempontok</span><span class="sxs-lookup"><span data-stu-id="43d1a-245">Manageability considerations</span></span>

<span data-ttu-id="43d1a-246">A fejlesztési és üzemeltetési csapatnak a következő feladatok elvégzésére kell felkészülnie:</span><span class="sxs-lookup"><span data-stu-id="43d1a-246">DevOps staff should be prepared to perform the following tasks:</span></span>

- <span data-ttu-id="43d1a-247">Az összevonási kiszolgálók kezelése, beleértve az AD FS-farm, az összevonási kiszolgálókhoz tartozó megbízhatósági házirendek és az összevonási szolgáltatások által használt tanúsítványok kezelését.</span><span class="sxs-lookup"><span data-stu-id="43d1a-247">Managing the federation servers, including managing the AD FS farm, managing trust policy on the federation servers, and managing the certificates used by the federation services.</span></span>
- <span data-ttu-id="43d1a-248">A WAP-kiszolgálók kezelése, beleértve a WAP-farmok és a tanúsítványok kezelését.</span><span class="sxs-lookup"><span data-stu-id="43d1a-248">Managing the WAP servers including managing the WAP farm and certificates.</span></span>
- <span data-ttu-id="43d1a-249">A webalkalmazások kezelése, beleértve a függő entitások, a hitelesítési módszerek és a jogcímleképezések konfigurálását.</span><span class="sxs-lookup"><span data-stu-id="43d1a-249">Managing web applications including configuring relying parties, authentication methods, and claims mappings.</span></span>
- <span data-ttu-id="43d1a-250">Az AD FS-összetevők biztonsági mentése.</span><span class="sxs-lookup"><span data-stu-id="43d1a-250">Backing up AD FS components.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="43d1a-251">Biztonsági szempontok</span><span class="sxs-lookup"><span data-stu-id="43d1a-251">Security considerations</span></span>

<span data-ttu-id="43d1a-252">Az AD FS HTTPS-t használ, ezért ügyeljen arra, hogy az NSG-szabályok az alhálózat, amely tartalmazza a webes szint virtuális gépek engedély HTTPS-kéréseket.</span><span class="sxs-lookup"><span data-stu-id="43d1a-252">AD FS uses HTTPS, so make sure that the NSG rules for the subnet containing the web tier VMs permit HTTPS requests.</span></span> <span data-ttu-id="43d1a-253">Ezek a kérelmek származhatnak a helyszíni hálózatról, a webes, az üzleti vagy az adatréteget tartalmazó alhálózatokról, a privát vagy a nyilvános DMZ-ről vagy az AD FS-kiszolgálókat tartalmazó alhálózatról.</span><span class="sxs-lookup"><span data-stu-id="43d1a-253">These requests can originate from the on-premises network, the subnets containing the web tier, business tier, data tier, private DMZ, public DMZ, and the subnet containing the AD FS servers.</span></span>

<span data-ttu-id="43d1a-254">Akadályozza meg az AD FS-kiszolgálók közvetlen elérését az internetről.</span><span class="sxs-lookup"><span data-stu-id="43d1a-254">Prevent direct exposure of the AD FS servers to the Internet.</span></span> <span data-ttu-id="43d1a-255">Az AD FS-kiszolgálók tartományhoz kapcsolt számítógépek, amelyek teljes körű engedéllyel rendelkeznek a biztonsági jogkivonatok kiállításához.</span><span class="sxs-lookup"><span data-stu-id="43d1a-255">AD FS servers are domain-joined computers that have full authorization to grant security tokens.</span></span> <span data-ttu-id="43d1a-256">Ha egy kiszolgáló sérül, a rosszindulatú felhasználók teljes körű hozzáférési jogkivonatokat bocsáthatnak ki az AD FS által védett összes webalkalmazáshoz és összevont kiszolgálóhoz.</span><span class="sxs-lookup"><span data-stu-id="43d1a-256">If a server is compromised, a malicious user can issue full access tokens to all web applications and to all federation servers that are protected by AD FS.</span></span> <span data-ttu-id="43d1a-257">Ha a rendszernek olyan kérelmeket is kezelnie kell, amelyek külső, nem megbízható partneroldalakról kapcsolódó felhasználóktól származnak, ezekhez a kérelmekhez használjon WAP-kiszolgálókat.</span><span class="sxs-lookup"><span data-stu-id="43d1a-257">If your system must handle requests from external users not connecting from trusted partner sites, use WAP servers to handle these requests.</span></span> <span data-ttu-id="43d1a-258">További információkért lásd: [Összevonási kiszolgálóproxy elhelyezése][where-to-place-an-fs-proxy].</span><span class="sxs-lookup"><span data-stu-id="43d1a-258">For more information, see [Where to Place a Federation Server Proxy][where-to-place-an-fs-proxy].</span></span>

<span data-ttu-id="43d1a-259">Az AD FS- és a WAP-kiszolgálókat különálló alhálózatokban helyezze el, saját tűzfallal.</span><span class="sxs-lookup"><span data-stu-id="43d1a-259">Place AD FS servers and WAP servers in separate subnets with their own firewalls.</span></span> <span data-ttu-id="43d1a-260">A tűzfalszabályok meghatározásához használhat NSG-szabályokat.</span><span class="sxs-lookup"><span data-stu-id="43d1a-260">You can use NSG rules to define firewall rules.</span></span> <span data-ttu-id="43d1a-261">Az összes tűzfalnak engedélyezni kell az adatforgalmat a 443-as (HTTPS) porton.</span><span class="sxs-lookup"><span data-stu-id="43d1a-261">All firewalls should allow traffic on port 443 (HTTPS).</span></span>

<span data-ttu-id="43d1a-262">Korlátozza a közvetlen bejelentkezést az AD FS- és a WAP-kiszolgálókra.</span><span class="sxs-lookup"><span data-stu-id="43d1a-262">Restrict direct sign in access to the AD FS and WAP servers.</span></span> <span data-ttu-id="43d1a-263">Csak a fejlesztési és üzemeltetési csapat számára legyen lehetséges a kapcsolódás.</span><span class="sxs-lookup"><span data-stu-id="43d1a-263">Only DevOps staff should be able to connect.</span></span> <span data-ttu-id="43d1a-264">A WAP-kiszolgálókat ne csatlakoztassa a tartományhoz.</span><span class="sxs-lookup"><span data-stu-id="43d1a-264">Do not join the WAP servers to the domain.</span></span>

<span data-ttu-id="43d1a-265">Érdemes lehet virtuális hálózati készülékeket használni, amelyek naplózzák a részletes információkat a virtuális hálózat peremén átmenő forgalomról.</span><span class="sxs-lookup"><span data-stu-id="43d1a-265">Consider using a set of network virtual appliances that logs detailed information on traffic traversing the edge of your virtual network for auditing purposes.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="43d1a-266">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="43d1a-266">Deploy the solution</span></span>

<span data-ttu-id="43d1a-267">Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][github].</span><span class="sxs-lookup"><span data-stu-id="43d1a-267">A deployment for this architecture is available on [GitHub][github].</span></span> <span data-ttu-id="43d1a-268">Vegye figyelembe, hogy a teljes üzembe helyezés eltarthat akár két órát, amely tartalmazza a VPN gateway létrehozása, és futtatja a parancsfájlokat, amelyek az Active Directory és az AD FS konfigurálása.</span><span class="sxs-lookup"><span data-stu-id="43d1a-268">Note that the entire deployment can take up to two hours, which includes creating the VPN gateway and running the scripts that configure Active Directory and AD FS.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="43d1a-269">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="43d1a-269">Prerequisites</span></span>

1. <span data-ttu-id="43d1a-270">Klónozza, ágaztassa vagy töltse le a zip-fájlját a [GitHub-adattár](https://github.com/mspnp/identity-reference-architectures).</span><span class="sxs-lookup"><span data-stu-id="43d1a-270">Clone, fork, or download the zip file for the [GitHub repository](https://github.com/mspnp/identity-reference-architectures).</span></span>

1. <span data-ttu-id="43d1a-271">Telepítse az [Azure CLI 2.0-t.](/cli/azure/install-azure-cli?view=azure-cli-latest)</span><span class="sxs-lookup"><span data-stu-id="43d1a-271">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

1. <span data-ttu-id="43d1a-272">Telepítse [az Azure építőelemei](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm-csomagot.</span><span class="sxs-lookup"><span data-stu-id="43d1a-272">Install the [Azure building blocks](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm package.</span></span>

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

1. <span data-ttu-id="43d1a-273">Jelentkezzen be Azure-fiókjába egy parancssorból, Bash-parancssorból vagy PowerShell-parancssorból a következő módon:</span><span class="sxs-lookup"><span data-stu-id="43d1a-273">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="43d1a-274">A szimulált helyszíni adatközpont üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="43d1a-274">Deploy the simulated on-premises datacenter</span></span>

1. <span data-ttu-id="43d1a-275">Keresse meg a `adfs` mappájában, a GitHub-adattárban.</span><span class="sxs-lookup"><span data-stu-id="43d1a-275">Navigate to the `adfs` folder of the GitHub repository.</span></span>

1. <span data-ttu-id="43d1a-276">Nyissa meg az `onprem.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="43d1a-276">Open the `onprem.json` file.</span></span> <span data-ttu-id="43d1a-277">Keresse meg a példányok `adminPassword`, `Password`, és `SafeModeAdminPassword` frissíteniük a jelszavakat, és.</span><span class="sxs-lookup"><span data-stu-id="43d1a-277">Search for instances of `adminPassword`, `Password`, and `SafeModeAdminPassword` and update the passwords.</span></span>

1. <span data-ttu-id="43d1a-278">Futtassa a következő parancsot, és várjon, amíg a telepítés befejezéséhez:</span><span class="sxs-lookup"><span data-stu-id="43d1a-278">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-infrastructure"></a><span data-ttu-id="43d1a-279">Az Azure-infrastruktúra üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="43d1a-279">Deploy the Azure infrastructure</span></span>

1. <span data-ttu-id="43d1a-280">Nyissa meg az `azure.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="43d1a-280">Open the `azure.json` file.</span></span>  <span data-ttu-id="43d1a-281">Keresse meg példányait `adminPassword` és `Password` és adja meg a jelszavak adatait.</span><span class="sxs-lookup"><span data-stu-id="43d1a-281">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

1. <span data-ttu-id="43d1a-282">Futtassa a következő parancsot, és várjon, amíg a telepítés befejezéséhez:</span><span class="sxs-lookup"><span data-stu-id="43d1a-282">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p azure.json --deploy
    ```

### <a name="set-up-the-ad-fs-farm"></a><span data-ttu-id="43d1a-283">Állítsa be az AD FS-farm</span><span class="sxs-lookup"><span data-stu-id="43d1a-283">Set up the AD FS farm</span></span>

1. <span data-ttu-id="43d1a-284">Nyissa meg az `adfs-farm-first.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="43d1a-284">Open the `adfs-farm-first.json` file.</span></span>  <span data-ttu-id="43d1a-285">Keresse meg `AdminPassword` , és cserélje le az alapértelmezett jelszót.</span><span class="sxs-lookup"><span data-stu-id="43d1a-285">Search for `AdminPassword` and replace the default password.</span></span>

1. <span data-ttu-id="43d1a-286">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="43d1a-286">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-first.json --deploy
    ```

1. <span data-ttu-id="43d1a-287">Nyissa meg az `adfs-farm-rest.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="43d1a-287">Open the `adfs-farm-rest.json` file.</span></span>  <span data-ttu-id="43d1a-288">Keresse meg `AdminPassword` , és cserélje le az alapértelmezett jelszót.</span><span class="sxs-lookup"><span data-stu-id="43d1a-288">Search for `AdminPassword` and replace the default password.</span></span>

1. <span data-ttu-id="43d1a-289">Futtassa a következő parancsot, és várjon, amíg a telepítés befejezéséhez:</span><span class="sxs-lookup"><span data-stu-id="43d1a-289">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-rest.json --deploy
    ```

### <a name="configure-ad-fs-part-1"></a><span data-ttu-id="43d1a-290">Konfigurálhatja az AD FS (1. rész)</span><span class="sxs-lookup"><span data-stu-id="43d1a-290">Configure AD FS (part 1)</span></span>

1. <span data-ttu-id="43d1a-291">Nevű virtuális Gépet egy távoli asztal munkamenetet nyit meg `ra-adfs-jb-vm1`, azaz a jumpboxot virtuális Gépet.</span><span class="sxs-lookup"><span data-stu-id="43d1a-291">Open a remote desktop session to the VM named `ra-adfs-jb-vm1`, which is the jumpbox VM.</span></span> <span data-ttu-id="43d1a-292">Felhasználónév: `testuser`.</span><span class="sxs-lookup"><span data-stu-id="43d1a-292">The user name is `testuser`.</span></span>

1. <span data-ttu-id="43d1a-293">A jumpbox, nyissa meg a nevű virtuális Gépet egy távoli asztali munkamenetet `ra-adfs-proxy-vm1`.</span><span class="sxs-lookup"><span data-stu-id="43d1a-293">From the jumpbox, open a remote desktop session to the VM named `ra-adfs-proxy-vm1`.</span></span> <span data-ttu-id="43d1a-294">A magánhálózati IP-cím 10.0.6.4.</span><span class="sxs-lookup"><span data-stu-id="43d1a-294">The private IP address is 10.0.6.4.</span></span>

1. <span data-ttu-id="43d1a-295">A távoli asztali munkamenet, futtassa a [PowerShell ISE-ben](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span><span class="sxs-lookup"><span data-stu-id="43d1a-295">From this remote desktop session, run the [PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span></span>

1. <span data-ttu-id="43d1a-296">A PowerShell-lel keresse meg a következő könyvtárban:</span><span class="sxs-lookup"><span data-stu-id="43d1a-296">In PowerShell, navigate to the following directory:</span></span>

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. <span data-ttu-id="43d1a-297">A parancsfájl panelen illessze be a következő kódot, és futtassa:</span><span class="sxs-lookup"><span data-stu-id="43d1a-297">Paste the following code into a script pane and run it:</span></span>

    ```powershell
    . .\adfs-webproxy.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxyApp -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxyApp
    ```

    <span data-ttu-id="43d1a-298">Jelenleg a `Get-Credential` kéri, adja meg a jelszót, a központi telepítési alkalmazásparaméter-fájlt adott meg.</span><span class="sxs-lookup"><span data-stu-id="43d1a-298">At the `Get-Credential` prompt, enter the password that you specified in the deployment parameter file.</span></span>

1. <span data-ttu-id="43d1a-299">Futtassa a következő parancsot a állapotának figyelése a [DSC](/powershell/dsc/overview/overview) konfiguráció:</span><span class="sxs-lookup"><span data-stu-id="43d1a-299">Run the following command to monitor the progress of the [DSC](/powershell/dsc/overview/overview) configuration:</span></span>

    ```powershell
    Get-DscConfigurationStatus
    ```

    <span data-ttu-id="43d1a-300">Konzisztencia elérni több percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="43d1a-300">It can take several minutes to reach consistency.</span></span> <span data-ttu-id="43d1a-301">Ebben az időszakban előfordulhat, hogy hibába ütközik, a parancsban.</span><span class="sxs-lookup"><span data-stu-id="43d1a-301">During this time, you may see errors from the command.</span></span> <span data-ttu-id="43d1a-302">Ha a konfiguráció sikeres, a kimenet az alábbihoz hasonlóan kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="43d1a-302">When the configuration succeeds, the output should look similar to the following:</span></span>

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

### <a name="configure-ad-fs-part-2"></a><span data-ttu-id="43d1a-303">Konfigurálhatja az AD FS (2. rész)</span><span class="sxs-lookup"><span data-stu-id="43d1a-303">Configure AD FS (part 2)</span></span>

1. <span data-ttu-id="43d1a-304">A jumpbox, nyissa meg a nevű virtuális Gépet egy távoli asztali munkamenetet `ra-adfs-proxy-vm2`.</span><span class="sxs-lookup"><span data-stu-id="43d1a-304">From the jumpbox, open a remote desktop session to the VM named `ra-adfs-proxy-vm2`.</span></span> <span data-ttu-id="43d1a-305">A magánhálózati IP-cím 10.0.6.5.</span><span class="sxs-lookup"><span data-stu-id="43d1a-305">The private IP address is 10.0.6.5.</span></span>

1. <span data-ttu-id="43d1a-306">A távoli asztali munkamenet, futtassa a [PowerShell ISE-ben](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span><span class="sxs-lookup"><span data-stu-id="43d1a-306">From this remote desktop session, run the [PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span></span>

1. <span data-ttu-id="43d1a-307">Keresse meg a következő könyvtárban:</span><span class="sxs-lookup"><span data-stu-id="43d1a-307">Navigate to the following directory:</span></span>

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. <span data-ttu-id="43d1a-308">A parancsfájl panelen és futtassa a parancsfájlt a következő elmúlt:</span><span class="sxs-lookup"><span data-stu-id="43d1a-308">Past the following in a script pane and run the script:</span></span>

    ```powershell
    . .\adfs-webproxy-rest.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxy -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxy
    ```

    <span data-ttu-id="43d1a-309">Jelenleg a `Get-Credential` kéri, adja meg a jelszót, a központi telepítési alkalmazásparaméter-fájlt adott meg.</span><span class="sxs-lookup"><span data-stu-id="43d1a-309">At the `Get-Credential` prompt, enter the password that you specified in the deployment parameter file.</span></span>

1. <span data-ttu-id="43d1a-310">A következő parancsot a DSC-konfiguráció állapotának figyelése:</span><span class="sxs-lookup"><span data-stu-id="43d1a-310">Run the following command to monitor the progress of the DSC configuration:</span></span>

    ```powershell
    Get-DscConfigurationStatus
    ```

    <span data-ttu-id="43d1a-311">Konzisztencia elérni több percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="43d1a-311">It can take several minutes to reach consistency.</span></span> <span data-ttu-id="43d1a-312">Ebben az időszakban előfordulhat, hogy hibába ütközik, a parancsban.</span><span class="sxs-lookup"><span data-stu-id="43d1a-312">During this time, you may see errors from the command.</span></span> <span data-ttu-id="43d1a-313">Ha a konfiguráció sikeres, a kimenet az alábbihoz hasonlóan kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="43d1a-313">When the configuration succeeds, the output should look similar to the following:</span></span>

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

    <span data-ttu-id="43d1a-314">Egyes esetekben ez DSC sikertelen lesz.</span><span class="sxs-lookup"><span data-stu-id="43d1a-314">Sometimes this DSC fails.</span></span> <span data-ttu-id="43d1a-315">Ha az állapot ellenőrzése látható `Status=Failure` és `Type=Consistency`, próbálja meg újra futtatni a 4. lépés.</span><span class="sxs-lookup"><span data-stu-id="43d1a-315">If the status check shows `Status=Failure` and `Type=Consistency`, try re-running step 4.</span></span>

### <a name="sign-into-ad-fs"></a><span data-ttu-id="43d1a-316">Jelentkezzen be az Active Directory összevonási szolgáltatások</span><span class="sxs-lookup"><span data-stu-id="43d1a-316">Sign into AD FS</span></span>

1. <span data-ttu-id="43d1a-317">A jumpbox, nyissa meg a nevű virtuális Gépet egy távoli asztali munkamenetet `ra-adfs-adfs-vm1`.</span><span class="sxs-lookup"><span data-stu-id="43d1a-317">From the jumpbox, open a remote desktop session to the VM named `ra-adfs-adfs-vm1`.</span></span> <span data-ttu-id="43d1a-318">A magánhálózati IP-cím 10.0.5.4.</span><span class="sxs-lookup"><span data-stu-id="43d1a-318">The private IP address is 10.0.5.4.</span></span>

1. <span data-ttu-id="43d1a-319">Kövesse a [az oldal Idp-Intiated bejelentkezés engedélyezése](/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-initiatedsignon#enable-the-idp-intiated-sign-on-page) ahhoz, hogy a bejelentkezési lapot.</span><span class="sxs-lookup"><span data-stu-id="43d1a-319">Follow the steps in [Enable the Idp-Intiated Sign on page](/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-initiatedsignon#enable-the-idp-intiated-sign-on-page) to enable the sign-on page.</span></span>

1. <span data-ttu-id="43d1a-320">A jump boxon keresse meg a `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm`.</span><span class="sxs-lookup"><span data-stu-id="43d1a-320">From the jump box, browse to `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm`.</span></span> <span data-ttu-id="43d1a-321">Egy tanúsítványt, hogy figyelmen kívül hagyhatja ezt a vizsgálatot a figyelmeztetés jelenhet meg.</span><span class="sxs-lookup"><span data-stu-id="43d1a-321">You may receive a certificate warning that you can ignore for this test.</span></span>

1. <span data-ttu-id="43d1a-322">Győződjön meg arról, hogy a Contoso Corporation bejelentkezési oldala jelenik meg.</span><span class="sxs-lookup"><span data-stu-id="43d1a-322">Verify that the Contoso Corporation sign-in page appears.</span></span> <span data-ttu-id="43d1a-323">Jelentkezzen be, **contoso\testuser**.</span><span class="sxs-lookup"><span data-stu-id="43d1a-323">Sign in as **contoso\testuser**.</span></span>

<!-- links -->
[extending-ad-to-azure]: adds-extend-domain.md

[vm-recommendations]: ../virtual-machines-windows/single-vm.md
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md
[hybrid-azure-on-prem-vpn]: ../hybrid-networking/vpn.md

[azure-cli]: /azure/azure-resource-manager/xplat-cli-azure-resource-manager
[DRS]: https://technet.microsoft.com/library/dn280945.aspx
[where-to-place-an-fs-proxy]: https://technet.microsoft.com/library/dd807048.aspx
[ADDRS]: https://technet.microsoft.com/library/dn486831.aspx
[plan-your-adfs-deployment]: https://msdn.microsoft.com/library/azure/dn151324.aspx
[ad_network_recommendations]: #network_configuration_recommendations_for_AD_DS_VMs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[create_service_account_for_adfs_farm]: https://technet.microsoft.com/library/dd807078.aspx
[adfs-configuration-database]: https://technet.microsoft.com/library/ee913581(v=ws.11).aspx
[active-directory-federation-services]: /windows-server/identity/active-directory-federation-services
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  /windows-server/identity/ad-fs/deployment/deploying-a-federation-server-farm
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: /powershell/azure/overview
[aad]: /azure/active-directory/
[aadb2c]: /azure/active-directory-b2c/
[adfs-intro]: /azure/active-directory/hybrid/whatis-hybrid-identity
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
