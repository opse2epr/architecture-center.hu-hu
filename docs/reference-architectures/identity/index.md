---
title: Helyszíni Active Directory integrálása az Azure-ban
titleSuffix: Azure Reference Architectures
description: Referenciaarchitektúrák összehasonlítása a helyszíni Active Directory Azure-ban való integrálásához.
ms.date: 07/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: 'seodec18, identity'
---

# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a><span data-ttu-id="3f869-103">Válasszon egy megoldást a helyszíni Active Directory Azure-ban való integráláshoz.</span><span class="sxs-lookup"><span data-stu-id="3f869-103">Choose a solution for integrating on-premises Active Directory with Azure</span></span>

<span data-ttu-id="3f869-104">Ez a cikk a helyszíni Active Directory (AD) környezet Azure-hálózattal való integrálásának lehetőségeit hasonlítja össze.</span><span class="sxs-lookup"><span data-stu-id="3f869-104">This article compares options for integrating your on-premises Active Directory (AD) environment with an Azure network.</span></span> <span data-ttu-id="3f869-105">Mindegyik lehetőséghez elérhető egy referenciaarchitektúra.</span><span class="sxs-lookup"><span data-stu-id="3f869-105">For each option, a more detailed reference architecture is available.</span></span>

<span data-ttu-id="3f869-106">Számos vállalat használja az Active Directory Domain Services (AD DS) szolgáltatásokat a felhasználókhoz, számítógépekhez, alkalmazásokhoz vagy egyéb, biztonsági határral védett erőforrásokhoz társított identitások hitelesítésére.</span><span class="sxs-lookup"><span data-stu-id="3f869-106">Many organizations use Active Directory Domain Services (AD DS) to authenticate identities associated with users, computers, applications, or other resources that are included in a security boundary.</span></span> <span data-ttu-id="3f869-107">A címtár- és identitásszolgáltatások általában helyszíni üzemeltetésűek, ám ha az alkalmazás üzemeltetése részben a helyszínen, részben pedig az Azure-ban történik, akkor késés léphet fel a hitelesítési kérések az Azure-ból a helyszínre történő küldése során.</span><span class="sxs-lookup"><span data-stu-id="3f869-107">Directory and identity services are typically hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, there may be latency sending authentication requests from Azure back to on-premises.</span></span> <span data-ttu-id="3f869-108">A késés csökkenthető a címtár- és identitásszolgáltatások Azure-beli implementálásával.</span><span class="sxs-lookup"><span data-stu-id="3f869-108">Implementing directory and identity services in Azure can reduce this latency.</span></span>

<span data-ttu-id="3f869-109">Az Azure két megoldást kínál a címtár- és identitásszolgáltatások Azure-beli implementációjához:</span><span class="sxs-lookup"><span data-stu-id="3f869-109">Azure provides two solutions for implementing directory and identity services in Azure:</span></span>

- <span data-ttu-id="3f869-110">Az [Azure AD] [ azure-active-directory] használatával létrehozhat egy Active Directory-tartományt a felhőben, és csatlakoztathatja azt a helyszíni Active Directory-tartományhoz.</span><span class="sxs-lookup"><span data-stu-id="3f869-110">Use [Azure AD][azure-active-directory] to create an Active Directory domain in the cloud and connect it to your on-premises Active Directory domain.</span></span> <span data-ttu-id="3f869-111">[Az Azure AD Connect] [ azure-ad-connect] integrálja a helyszíni címtárakat az Azure AD-vel.</span><span class="sxs-lookup"><span data-stu-id="3f869-111">[Azure AD Connect][azure-ad-connect] integrates your on-premises directories with Azure AD.</span></span>

- <span data-ttu-id="3f869-112">Kiterjesztheti meglévő helyszíni Active Directory-infrastruktúráját az Azure-ba, ha egy olyan virtuális gépet helyez üzembe az Azure-ban, amely tartományvezérlőként futtatja az AD DS-t.</span><span class="sxs-lookup"><span data-stu-id="3f869-112">Extend your existing on-premises Active Directory infrastructure to Azure, by deploying a VM in Azure that runs AD DS as a domain controller.</span></span> <span data-ttu-id="3f869-113">Ez az architektúra általában akkor használatos, amikor a helyszíni hálózatot és az Azure-beli virtuális hálózatot (VNet) VPN- vagy ExpressRoute-kapcsolat köti össze.</span><span class="sxs-lookup"><span data-stu-id="3f869-113">This architecture is more common when the on-premises network and the Azure virtual network (VNet) are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="3f869-114">Ennek az architektúrának több változata is lehetséges:</span><span class="sxs-lookup"><span data-stu-id="3f869-114">Several variations of this architecture are possible:</span></span>

  - <span data-ttu-id="3f869-115">Létrehozhat egy tartományt az Azure-ban, és csatlakoztathatja azt a helyszíni AD-erdőhöz.</span><span class="sxs-lookup"><span data-stu-id="3f869-115">Create a domain in Azure and join it to your on-premises AD forest.</span></span>
  - <span data-ttu-id="3f869-116">Létrehozhat egy külön erdőt az Azure-ban, amelyet a helyszíni erdő tartományai megbízhatónak tekintenek.</span><span class="sxs-lookup"><span data-stu-id="3f869-116">Create a separate forest in Azure that is trusted by domains in your on-premises forest.</span></span>
  - <span data-ttu-id="3f869-117">Replikálhatja az Active Directory összevonási szolgáltatások (AD FS) egy üzemelő példányát az Azure-ba.</span><span class="sxs-lookup"><span data-stu-id="3f869-117">Replicate an Active Directory Federation Services (AD FS) deployment to Azure.</span></span>

<span data-ttu-id="3f869-118">A következő szakaszokban a felsorolt lehetőségek részletes leírása szerepel.</span><span class="sxs-lookup"><span data-stu-id="3f869-118">The next sections describe each of these options in more detail.</span></span>

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a><span data-ttu-id="3f869-119">Helyszíni tartományok integrálása az Azure AD-be</span><span class="sxs-lookup"><span data-stu-id="3f869-119">Integrate your on-premises domains with Azure AD</span></span>

<span data-ttu-id="3f869-120">Az Azure Active Directory (Azure AD) használatával hozzon létre egy tartományt az Azure-ban, és csatlakoztassa azt egy helyszíni Active Directory-tartományhoz.</span><span class="sxs-lookup"><span data-stu-id="3f869-120">Use Azure Active Directory (Azure AD) to create a domain in Azure and link it to an on-premises AD domain.</span></span>

<span data-ttu-id="3f869-121">Az Azure AD-címtár nem a helyszíni címtár kiterjesztése, </span><span class="sxs-lookup"><span data-stu-id="3f869-121">The Azure AD directory is not an extension of an on-premises directory.</span></span> <span data-ttu-id="3f869-122">hanem egy ugyanazon objektumokat és identitásokat tartalmazó másolat.</span><span class="sxs-lookup"><span data-stu-id="3f869-122">Rather, it's a copy that contains the same objects and identities.</span></span> <span data-ttu-id="3f869-123">Az elemeken a helyszínen elvégzett módosításokat a rendszer átmásolja az Azure AD-be, az Azure AD-ben végrehajtott módosításokat azonban nem replikálja a helyszíni tartományra.</span><span class="sxs-lookup"><span data-stu-id="3f869-123">Changes made to these items on-premises are copied to Azure AD, but changes made in Azure AD are not replicated back to the on-premises domain.</span></span>

<span data-ttu-id="3f869-124">Az Azure AD-t egy helyszíni címtár használata nélkül is használhatja.</span><span class="sxs-lookup"><span data-stu-id="3f869-124">You can also use Azure AD without using an on-premises directory.</span></span> <span data-ttu-id="3f869-125">Ebben az esetben az Azure AD az azonosító adatok elsődleges forrásaként szolgál, és nem tartalmaz a helyszíni címtárból replikált adatokat.</span><span class="sxs-lookup"><span data-stu-id="3f869-125">In this case, Azure AD acts as the primary source of all identity information, rather than containing data replicated from an on-premises directory.</span></span>

<span data-ttu-id="3f869-126">**Előnyök**</span><span class="sxs-lookup"><span data-stu-id="3f869-126">**Benefits**</span></span>

- <span data-ttu-id="3f869-127">Nem kell fenntartania AD-infrastruktúrát a felhőben.</span><span class="sxs-lookup"><span data-stu-id="3f869-127">You don't need to maintain an AD infrastructure in the cloud.</span></span> <span data-ttu-id="3f869-128">Az Azure AD fenntartását és felügyeletét teljes mértékben a Microsoft végzi.</span><span class="sxs-lookup"><span data-stu-id="3f869-128">Azure AD is entirely managed and maintained by Microsoft.</span></span>
- <span data-ttu-id="3f869-129">Az Azure AD ugyanazokat az azonosító adatokat biztosítja, amelyek a helyszínen is elérhetők.</span><span class="sxs-lookup"><span data-stu-id="3f869-129">Azure AD provides the same identity information that is available on-premises.</span></span>
- <span data-ttu-id="3f869-130">A hitelesítés az Azure-ban is megtörténhet, ezáltal a külső alkalmazásoknak és a felhasználóknak ritkábban kell kapcsolatba lépniük a helyszíni tartománnyal.</span><span class="sxs-lookup"><span data-stu-id="3f869-130">Authentication can happen in Azure, reducing the need for external applications and users to contact the on-premises domain.</span></span>

<span data-ttu-id="3f869-131">**Problémák**</span><span class="sxs-lookup"><span data-stu-id="3f869-131">**Challenges**</span></span>

- <span data-ttu-id="3f869-132">Az identitásszolgáltatások a felhasználókra és a csoportokra korlátozódnak.</span><span class="sxs-lookup"><span data-stu-id="3f869-132">Identity services are limited to users and groups.</span></span> <span data-ttu-id="3f869-133">Nincs lehetőség a szolgáltatás- és számítógépes fiókok hitelesítésére.</span><span class="sxs-lookup"><span data-stu-id="3f869-133">There is no ability to authenticate service and computer accounts.</span></span>
- <span data-ttu-id="3f869-134">Az Azure AD-címtár folyamatos szinkronizálásához konfigurálnia kell a kapcsolatot a helyszíni tartománnyal.</span><span class="sxs-lookup"><span data-stu-id="3f869-134">You must configure connectivity with your on-premises domain to keep the Azure AD directory synchronized.</span></span>
- <span data-ttu-id="3f869-135">Előfordulhat, hogy az alkalmazásokat újra kell írni az Azure AD-n keresztüli hitelesítés engedélyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="3f869-135">Applications may need to be rewritten to enable authentication through Azure AD.</span></span>

<span data-ttu-id="3f869-136">**Referenciaarchitektúra**</span><span class="sxs-lookup"><span data-stu-id="3f869-136">**Reference architecture**</span></span>

- <span data-ttu-id="3f869-137">[Helyszíni Active Directory-tartományok integrálása az Azure Active Directoryval][aad]</span><span class="sxs-lookup"><span data-stu-id="3f869-137">[Integrate on-premises Active Directory domains with Azure Active Directory][aad]</span></span>

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a><span data-ttu-id="3f869-138">Helyszíni erdőhöz csatlakozó AD DS az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="3f869-138">AD DS in Azure joined to an on-premises forest</span></span>

<span data-ttu-id="3f869-139">Azure AD Domain Services- (AD DS) kiszolgálókat helyezhet üzembe az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3f869-139">Deploy AD Domain Services (AD DS) servers to Azure.</span></span> <span data-ttu-id="3f869-140">Létrehozhat egy tartományt az Azure-ban, és csatlakoztathatja azt a helyszíni AD-erdőhöz.</span><span class="sxs-lookup"><span data-stu-id="3f869-140">Create a domain in Azure and join it to your on-premises AD forest.</span></span>

<span data-ttu-id="3f869-141">Fontolja meg ezt a lehetőséget, ha az AD DS azon funkcióit szeretné használni, amelyek jelenleg nincsenek implementálva az Azure AD-ben.</span><span class="sxs-lookup"><span data-stu-id="3f869-141">Consider this option if you need to use AD DS features that are not currently implemented by Azure AD.</span></span>

<span data-ttu-id="3f869-142">**Előnyök**</span><span class="sxs-lookup"><span data-stu-id="3f869-142">**Benefits**</span></span>

- <span data-ttu-id="3f869-143">Hozzáférést biztosít a helyszínen is elérhető azonosító adatokhoz.</span><span class="sxs-lookup"><span data-stu-id="3f869-143">Provides access to the same identity information that is available on-premises.</span></span>
- <span data-ttu-id="3f869-144">Felhasználói, szolgáltatás- és számítógépes fiókok hitelesítését a helyszíni környezetben és az Azure-ban is elvégezheti.</span><span class="sxs-lookup"><span data-stu-id="3f869-144">You can authenticate user, service, and computer accounts on-premises and in Azure.</span></span>
- <span data-ttu-id="3f869-145">Nincs szükség külön AD-erdő kezelésére.</span><span class="sxs-lookup"><span data-stu-id="3f869-145">You don't need to manage a separate AD forest.</span></span> <span data-ttu-id="3f869-146">Az Azure-beli tartomány a helyi erdőhöz is tartozhat.</span><span class="sxs-lookup"><span data-stu-id="3f869-146">The domain in Azure can belong to the on-premises forest.</span></span>
- <span data-ttu-id="3f869-147">A helyszíni csoportszabályzat-objektumok által meghatározott csoportszabályzatot az Azure-beli tartományra is alkalmazhatja.</span><span class="sxs-lookup"><span data-stu-id="3f869-147">You can apply group policy defined by on-premises Group Policy Objects to the domain in Azure.</span></span>

<span data-ttu-id="3f869-148">**Problémák**</span><span class="sxs-lookup"><span data-stu-id="3f869-148">**Challenges**</span></span>

- <span data-ttu-id="3f869-149">Saját magának kell elvégeznie az AD DS-kiszolgálók és a tartomány üzembe helyezését és kezelését a felhőben.</span><span class="sxs-lookup"><span data-stu-id="3f869-149">You must deploy and manage your own AD DS servers and domain in the cloud.</span></span>
- <span data-ttu-id="3f869-150">A felhőbeli tartománykiszolgálók és helyszínen futó kiszolgálók között előfordulhat némi szinkronizálási késés.</span><span class="sxs-lookup"><span data-stu-id="3f869-150">There may be some synchronization latency between the domain servers in the cloud and the servers running on-premises.</span></span>

<span data-ttu-id="3f869-151">**Referenciaarchitektúra**</span><span class="sxs-lookup"><span data-stu-id="3f869-151">**Reference architecture**</span></span>

- <span data-ttu-id="3f869-152">[Az Azure Active Directory Domain Services (AD DS) kiterjesztése az Azure-ra][ad-ds]</span><span class="sxs-lookup"><span data-stu-id="3f869-152">[Extend Active Directory Domain Services (AD DS) to Azure][ad-ds]</span></span>

## <a name="ad-ds-in-azure-with-a-separate-forest"></a><span data-ttu-id="3f869-153">Külön erdővel rendelkező AD DS az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="3f869-153">AD DS in Azure with a separate forest</span></span>

<span data-ttu-id="3f869-154">Helyezzen üzembe AD Domain Services- (AD DS) kiszolgálókat az Azure-ban, de hozzon létre egy külön Active Directory-[erdőt][ad-forest-defn], amely elkülönül a helyszíni erdőtől.</span><span class="sxs-lookup"><span data-stu-id="3f869-154">Deploy AD Domain Services (AD DS) servers to Azure, but create a separate Active Directory [forest][ad-forest-defn] that is separate from the on-premises forest.</span></span> <span data-ttu-id="3f869-155">Ezt az erdőt a helyszíni erdő tartományai megbízhatónak tekintik.</span><span class="sxs-lookup"><span data-stu-id="3f869-155">This forest is trusted by domains in your on-premises forest.</span></span>

<span data-ttu-id="3f869-156">Az architektúra gyakori használati módjai például a felhőben tárolt objektumok és identitások biztonsági elkülönítésének fenntartása, illetve az egyes tartományok migrálása a helyszínről a felhőbe.</span><span class="sxs-lookup"><span data-stu-id="3f869-156">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="3f869-157">**Előnyök**</span><span class="sxs-lookup"><span data-stu-id="3f869-157">**Benefits**</span></span>

- <span data-ttu-id="3f869-158">Helyszíni és külön, csak az Azure-hoz tartozó identitásokat is megvalósíthat.</span><span class="sxs-lookup"><span data-stu-id="3f869-158">You can implement on-premises identities and separate Azure-only identities.</span></span>
- <span data-ttu-id="3f869-159">Nem kell replikációt végezni a helyszíni AD-erdőből az Azure-ba.</span><span class="sxs-lookup"><span data-stu-id="3f869-159">You don't need to replicate from the on-premises AD forest to Azure.</span></span>

<span data-ttu-id="3f869-160">**Problémák**</span><span class="sxs-lookup"><span data-stu-id="3f869-160">**Challenges**</span></span>

- <span data-ttu-id="3f869-161">A helyszíni identitások Azure-beli hitelesítéshez több, a helyszíni AD-kiszolgálókhoz való hálózati ugrásra van szükség.</span><span class="sxs-lookup"><span data-stu-id="3f869-161">Authentication within Azure for on-premises identities requires extra network hops to the on-premises AD servers.</span></span>
- <span data-ttu-id="3f869-162">Saját AD DS-kiszolgálókat és erdőt kell üzembe helyeznie a felhőben, és az erdők között megfelelő megbízhatósági kapcsolatokat kell létesítenie.</span><span class="sxs-lookup"><span data-stu-id="3f869-162">You must deploy your own AD DS servers and forest in the cloud, and establish the appropriate trust relationships between forests.</span></span>

<span data-ttu-id="3f869-163">**Referenciaarchitektúra**</span><span class="sxs-lookup"><span data-stu-id="3f869-163">**Reference architecture**</span></span>

- <span data-ttu-id="3f869-164">[Active Directory Domain Services- (AD DS-) erőforráserdő létrehozása az Azure-ban][ad-ds-forest]</span><span class="sxs-lookup"><span data-stu-id="3f869-164">[Create an Active Directory Domain Services (AD DS) resource forest in Azure][ad-ds-forest]</span></span>

## <a name="extend-ad-fs-to-azure"></a><span data-ttu-id="3f869-165">Az AD FS kiterjesztése az Azure-ra</span><span class="sxs-lookup"><span data-stu-id="3f869-165">Extend AD FS to Azure</span></span>

<span data-ttu-id="3f869-166">Replikáljon az Active Directory összevonási szolgáltatások (AD FS) egy üzemelő példányát az Azure-ba az Azure-beli összetevők összevont hitelesítéséhez és engedélyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="3f869-166">Replicate an Active Directory Federation Services (AD FS) deployment to Azure, to perform federated authentication and authorization for components running in Azure.</span></span>

<span data-ttu-id="3f869-167">Az architektúra gyakori használati módjai a következők:</span><span class="sxs-lookup"><span data-stu-id="3f869-167">Typical uses for this architecture:</span></span>

- <span data-ttu-id="3f869-168">Partnerszervezetek felhasználóinak hitelesítése és engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="3f869-168">Authenticate and authorize users from partner organizations.</span></span>
- <span data-ttu-id="3f869-169">A felhasználói hitelesítés engedélyezése a vállalati tűzfalon kívül futó webböngészőkből.</span><span class="sxs-lookup"><span data-stu-id="3f869-169">Allow users to authenticate from web browsers running outside of the organizational firewall.</span></span>
- <span data-ttu-id="3f869-170">A felhasználók hitelesített külső eszközökről, például mobileszközökről történő csatlakozásának engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="3f869-170">Allow users to connect from authorized external devices such as mobile devices.</span></span>

<span data-ttu-id="3f869-171">**Előnyök**</span><span class="sxs-lookup"><span data-stu-id="3f869-171">**Benefits**</span></span>

- <span data-ttu-id="3f869-172">Kihasználhatja a jogcímbarát alkalmazások előnyeit.</span><span class="sxs-lookup"><span data-stu-id="3f869-172">You can leverage claims-aware applications.</span></span>
- <span data-ttu-id="3f869-173">Lehetővé teszi, hogy megbízzon a külső partnerekben a hitelesítés tekintetében.</span><span class="sxs-lookup"><span data-stu-id="3f869-173">Provides the ability to trust external partners for authentication.</span></span>
- <span data-ttu-id="3f869-174">Nagy hitelesítésiprotokoll-készletekkel is kompatibilis.</span><span class="sxs-lookup"><span data-stu-id="3f869-174">Compatibility with large set of authentication protocols.</span></span>

<span data-ttu-id="3f869-175">**Problémák**</span><span class="sxs-lookup"><span data-stu-id="3f869-175">**Challenges**</span></span>

- <span data-ttu-id="3f869-176">Saját kezűleg kell üzembe helyeznie AD DS, AD FS és AD FS webalkalmazás-proxy kiszolgálóit az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3f869-176">You must deploy your own AD DS, AD FS, and AD FS Web Application Proxy servers in Azure.</span></span>
- <span data-ttu-id="3f869-177">Az architektúra konfigurálása bonyolult lehet.</span><span class="sxs-lookup"><span data-stu-id="3f869-177">This architecture can be complex to configure.</span></span>

<span data-ttu-id="3f869-178">**Referenciaarchitektúra**</span><span class="sxs-lookup"><span data-stu-id="3f869-178">**Reference architecture**</span></span>

- <span data-ttu-id="3f869-179">[Az Active Directory összevonási szolgáltatások (AD FS) kiterjesztése az Azure-ra][adfs]</span><span class="sxs-lookup"><span data-stu-id="3f869-179">[Extend Active Directory Federation Services (AD FS) to Azure][adfs]</span></span>

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: /windows/desktop/AD/forests
[adfs]: ./adfs.md

[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
