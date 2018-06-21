---
title: Válassza ki a helyszíni Active Directory integrálása az Azure a megoldást.
description: Összehasonlítja a helyszíni Active Directory integrálása Azure architektúrához hivatkozik.
ms.date: 04/06/2017
ms.openlocfilehash: 413a5463d90547197c4b6834d353b4ecf61483ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541657"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a><span data-ttu-id="61f99-103">Válassza ki a megoldást a helyszíni Active Directory integrálása az Azure-ral</span><span class="sxs-lookup"><span data-stu-id="61f99-103">Choose a solution for integrating on-premises Active Directory with Azure</span></span>

<span data-ttu-id="61f99-104">Ez a cikk összehasonlítja a helyszíni Active Directory (AD) környezetben integrálása az Azure-hálózat beállításait.</span><span class="sxs-lookup"><span data-stu-id="61f99-104">This article compares options for integrating your on-premises Active Directory (AD) environment with an Azure network.</span></span> <span data-ttu-id="61f99-105">Az egyes lehetőségek nyújtunk a referencia-architektúrában és a központilag telepíthető megoldás.</span><span class="sxs-lookup"><span data-stu-id="61f99-105">We provide a reference architecture and a deployable solution for each option.</span></span>

<span data-ttu-id="61f99-106">Számos szervezet használ [Active Directory tartományi szolgáltatások (AD DS)] [ active-directory-domain-services] hitelesítéshez társított felhasználók, számítógépek, alkalmazások vagy más erőforrásokat, amelyek szerepelnek a biztonsági azonosítók határ.</span><span class="sxs-lookup"><span data-stu-id="61f99-106">Many organizations use [Active Directory Domain Services (AD DS)][active-directory-domain-services] to authenticate identities associated with users, computers, applications, or other resources that are included in a security boundary.</span></span> <span data-ttu-id="61f99-107">Címtár- és identitáskezelési szolgáltatásainak használata általában üzemeltethető a helyszínen, de ha az alkalmazás üzemel részben helyszíni és részben az Azure-ban előfordulhat, küldő hitelesítési kérelmek az Azure-ból a helyszíni biztonsági késés.</span><span class="sxs-lookup"><span data-stu-id="61f99-107">Directory and identity services are typically hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, there may be latency sending authentication requests from Azure back to on-premises.</span></span> <span data-ttu-id="61f99-108">Címtár- és identitáskezelési szolgáltatásokat végrehajtása az Azure-ban csökkenti a késés.</span><span class="sxs-lookup"><span data-stu-id="61f99-108">Implementing directory and identity services in Azure can reduce this latency.</span></span>

<span data-ttu-id="61f99-109">Azure két megoldást nyújt a címtár- és identitáskezelési szolgáltatásokat végrehajtása az Azure-ban:</span><span class="sxs-lookup"><span data-stu-id="61f99-109">Azure provides two solutions for implementing directory and identity services in Azure:</span></span> 

* <span data-ttu-id="61f99-110">Használjon [az Azure AD] [ azure-active-directory] Active Directory-tartomány létrehozása a felhőben, és csatlakoztassa a helyszíni Active Directory-tartományhoz.</span><span class="sxs-lookup"><span data-stu-id="61f99-110">Use [Azure AD][azure-active-directory] to create an Active Directory domain in the cloud and connect it to your on-premises Active Directory domain.</span></span> <span data-ttu-id="61f99-111">[Az Azure AD Connect] [ azure-ad-connect] integrálható a helyszíni címtárakat az Azure AD.</span><span class="sxs-lookup"><span data-stu-id="61f99-111">[Azure AD Connect][azure-ad-connect] integrates your on-premises directories with Azure AD.</span></span>

* <span data-ttu-id="61f99-112">Az Azure-ba, a meglévő helyszíni Active Directory-infrastruktúra meghosszabbíthatja egy Azure-ban futó Active Directory tartományi szolgáltatások tartományvezérlő telepítését.</span><span class="sxs-lookup"><span data-stu-id="61f99-112">Extend your existing on-premises Active Directory infrastructure to Azure, by deploying a VM in Azure that runs AD DS as a domain controller.</span></span> <span data-ttu-id="61f99-113">Ez az architektúra napjainkban egyre általánosabbá, VPN vagy ExpressRoute kapcsolattal csatlakoztatott a helyszíni hálózat és az Azure virtuális hálózat (VNet).</span><span class="sxs-lookup"><span data-stu-id="61f99-113">This architecture is more common when the on-premises network and the Azure virtual network (VNet) are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="61f99-114">Ez az architektúra több változatait lehetségesek:</span><span class="sxs-lookup"><span data-stu-id="61f99-114">Several variations of this architecture are possible:</span></span> 

    - <span data-ttu-id="61f99-115">Hozzon létre egy tartományt az Azure-ban, és csatlakoztassa azt a helyszíni Active Directory-erdőben.</span><span class="sxs-lookup"><span data-stu-id="61f99-115">Create a domain in Azure and join it to your on-premises AD forest.</span></span>
    - <span data-ttu-id="61f99-116">Hozzon létre egy másik erdőben, amely megbízható-e a helyi erdőben lévő tartományokba Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="61f99-116">Create a separate forest in Azure that is trusted by domains in your on-premises forest.</span></span>
    - <span data-ttu-id="61f99-117">Az Active Directory összevonási szolgáltatások (AD FS) központi telepítése replikálása Azure-bA.</span><span class="sxs-lookup"><span data-stu-id="61f99-117">Replicate an Active Directory Federation Services (AD FS) deployment to Azure.</span></span> 

<span data-ttu-id="61f99-118">A következő szakaszok ismertetik részletesebben ezek közül.</span><span class="sxs-lookup"><span data-stu-id="61f99-118">The next sections describe each of these options in more detail.</span></span>

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a><span data-ttu-id="61f99-119">A helyszíni tartományok integrálása az Azure ad szolgáltatással</span><span class="sxs-lookup"><span data-stu-id="61f99-119">Integrate your on-premises domains with Azure AD</span></span>

<span data-ttu-id="61f99-120">Azure Active Directory (Azure AD) segítségével az Azure-tartomány létrehozása hozzákapcsolja azt egy a helyszíni Active Directory-tartománynak.</span><span class="sxs-lookup"><span data-stu-id="61f99-120">Use Azure Active Directory (Azure AD) to create a domain in Azure and link it to an on-premises AD domain.</span></span> 

<span data-ttu-id="61f99-121">Az Azure AD-címtár nincs kiterjesztése a helyszíni címtárba.</span><span class="sxs-lookup"><span data-stu-id="61f99-121">The Azure AD directory is not an extension of an on-premises directory.</span></span> <span data-ttu-id="61f99-122">Ehelyett egy másolatot, amely tartalmazza az ugyanazon objektumokat és identitások.</span><span class="sxs-lookup"><span data-stu-id="61f99-122">Rather, it's a copy that contains the same objects and identities.</span></span> <span data-ttu-id="61f99-123">Ezen elemek helyszíni módosításainak másolja az Azure AD, de a helyszíni tartománynév nem replikálja a Azure AD-ben végrehajtott módosításokat.</span><span class="sxs-lookup"><span data-stu-id="61f99-123">Changes made to these items on-premises are copied to Azure AD, but changes made in Azure AD are not replicated back to the on-premises domain.</span></span>

<span data-ttu-id="61f99-124">Az Azure AD egy helyszíni címtár használata nélkül is használható.</span><span class="sxs-lookup"><span data-stu-id="61f99-124">You can also use Azure AD without using an on-premises directory.</span></span> <span data-ttu-id="61f99-125">Ebben az esetben az Azure AD szolgáltatásba, illetve az elsődleges adatforrás összes azonosító adatok, nem pedig egy helyi könyvtárból replikált adatokat tartalmazó.</span><span class="sxs-lookup"><span data-stu-id="61f99-125">In this case, Azure AD acts as the primary source of all identity information, rather than containing data replicated from an on-premises directory.</span></span>


<span data-ttu-id="61f99-126">**Előnyei**</span><span class="sxs-lookup"><span data-stu-id="61f99-126">**Benefits**</span></span>

* <span data-ttu-id="61f99-127">Nem kell fenntartani egy AD-infrastruktúrát, a felhőben.</span><span class="sxs-lookup"><span data-stu-id="61f99-127">You don't need to maintain an AD infrastructure in the cloud.</span></span> <span data-ttu-id="61f99-128">Az Azure AD teljes mértékben felügyelt és a Microsoft által fenntartott.</span><span class="sxs-lookup"><span data-stu-id="61f99-128">Azure AD is entirely managed and maintained by Microsoft.</span></span>
* <span data-ttu-id="61f99-129">Az Azure AD elérhető helyszíni identitás adatokat biztosít.</span><span class="sxs-lookup"><span data-stu-id="61f99-129">Azure AD provides the same identity information that is available on-premises.</span></span>
* <span data-ttu-id="61f99-130">Hitelesítési akkor fordulhat elő, így a külső alkalmazások és a felhasználók csatlakozni tudnak a helyi tartomány, az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="61f99-130">Authentication can happen in Azure, reducing the need for external applications and users to contact the on-premises domain.</span></span>

<span data-ttu-id="61f99-131">**Kihívásai**</span><span class="sxs-lookup"><span data-stu-id="61f99-131">**Challenges**</span></span>

* <span data-ttu-id="61f99-132">Identitás-szolgáltatások a felhasználók és csoportok korlátozódnak.</span><span class="sxs-lookup"><span data-stu-id="61f99-132">Identity services are limited to users and groups.</span></span> <span data-ttu-id="61f99-133">Nincs nincs elvégezhessék a hitelesítést a szolgáltatás- és számítógépes fiókok.</span><span class="sxs-lookup"><span data-stu-id="61f99-133">There is no ability to authenticate service and computer accounts.</span></span>
* <span data-ttu-id="61f99-134">Konfigurálnia kell a kapcsolat a helyszíni tartománnyal tartani az Azure AD directory szinkronizálva.</span><span class="sxs-lookup"><span data-stu-id="61f99-134">You must configure connectivity with your on-premises domain to keep the Azure AD directory synchronized.</span></span> 
* <span data-ttu-id="61f99-135">Alkalmazások kell hitelesítést az Azure AD keresztül kell írni.</span><span class="sxs-lookup"><span data-stu-id="61f99-135">Applications may need to be rewritten to enable authentication through Azure AD.</span></span>

<span data-ttu-id="61f99-136">**[További tudnivalók...][aad]**</span><span class="sxs-lookup"><span data-stu-id="61f99-136">**[Read more...][aad]**</span></span>

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a><span data-ttu-id="61f99-137">Az Azure Active Directory tartományi szolgáltatások csatlakozik egy helyszíni erdőben</span><span class="sxs-lookup"><span data-stu-id="61f99-137">AD DS in Azure joined to an on-premises forest</span></span>

<span data-ttu-id="61f99-138">Azure AD tartományi szolgáltatások (AD DS) kiszolgálók üzembe.</span><span class="sxs-lookup"><span data-stu-id="61f99-138">Deploy AD Domain Services (AD DS) servers to Azure.</span></span> <span data-ttu-id="61f99-139">Hozzon létre egy tartományt az Azure-ban, és csatlakoztassa azt a helyszíni Active Directory-erdőben.</span><span class="sxs-lookup"><span data-stu-id="61f99-139">Create a domain in Azure and join it to your on-premises AD forest.</span></span> 

<span data-ttu-id="61f99-140">Fontolja meg ezt a beállítást, ha szeretné használni az Active Directory Tartományi szolgáltatások, amelyek jelenleg nem működik az Azure AD.</span><span class="sxs-lookup"><span data-stu-id="61f99-140">Consider this option if you need to use AD DS features that are not currently implemented by Azure AD.</span></span> 

<span data-ttu-id="61f99-141">**Előnyei**</span><span class="sxs-lookup"><span data-stu-id="61f99-141">**Benefits**</span></span>

* <span data-ttu-id="61f99-142">Rendelkezésre álló helyszíni identitás adatokat hozzáférést biztosít.</span><span class="sxs-lookup"><span data-stu-id="61f99-142">Provides access to the same identity information that is available on-premises.</span></span>
* <span data-ttu-id="61f99-143">Képes hitelesíteni a felhasználót, a szolgáltatás és a számítógép fiókok a helyszíni és az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="61f99-143">You can authenticate user, service, and computer accounts on-premises and in Azure.</span></span>
* <span data-ttu-id="61f99-144">Nem kell kezelni egy másik AD-erdőben.</span><span class="sxs-lookup"><span data-stu-id="61f99-144">You don't need to manage a separate AD forest.</span></span> <span data-ttu-id="61f99-145">A tartomány az Azure-ban a helyi erdőben is tartozhatnak.</span><span class="sxs-lookup"><span data-stu-id="61f99-145">The domain in Azure can belong to the on-premises forest.</span></span>
* <span data-ttu-id="61f99-146">A csoportházirend határozzák meg a helyi csoportházirend-objektumok a tartományhoz az Azure-ban is alkalmazhat.</span><span class="sxs-lookup"><span data-stu-id="61f99-146">You can apply group policy defined by on-premises Group Policy Objects to the domain in Azure.</span></span>

<span data-ttu-id="61f99-147">**Kihívásai**</span><span class="sxs-lookup"><span data-stu-id="61f99-147">**Challenges**</span></span>

* <span data-ttu-id="61f99-148">Kell telepíti és kezeli a saját Active Directory tartományi szolgáltatások-kiszolgálók és a tartományt a felhőben.</span><span class="sxs-lookup"><span data-stu-id="61f99-148">You must deploy and manage your own AD DS servers and domain in the cloud.</span></span>
* <span data-ttu-id="61f99-149">Előfordulhat, hogy egyes szinkronizálási várakozási ideje a tartománykiszolgálók a felhőben és helyszíni futtató kiszolgálók között.</span><span class="sxs-lookup"><span data-stu-id="61f99-149">There may be some synchronization latency between the domain servers in the cloud and the servers running on-premises.</span></span>

<span data-ttu-id="61f99-150">**[További tudnivalók...][ad-ds]**</span><span class="sxs-lookup"><span data-stu-id="61f99-150">**[Read more...][ad-ds]**</span></span>

## <a name="ad-ds-in-azure-with-a-separate-forest"></a><span data-ttu-id="61f99-151">Active Directory tartományi szolgáltatások az Azure-ban egy másik erdőben</span><span class="sxs-lookup"><span data-stu-id="61f99-151">AD DS in Azure with a separate forest</span></span>

<span data-ttu-id="61f99-152">Az Azure AD tartományi szolgáltatások (AD DS) kiszolgálók telepítése, de hozzon létre egy külön Active Directory [erdő] [ ad-forest-defn] , amely nem csatlakozik a helyi erdőben.</span><span class="sxs-lookup"><span data-stu-id="61f99-152">Deploy AD Domain Services (AD DS) servers to Azure, but create a separate Active Directory [forest][ad-forest-defn] that is separate from the on-premises forest.</span></span> <span data-ttu-id="61f99-153">Ez az erdő megbízhatónak a helyi erdőben lévő tartományokba.</span><span class="sxs-lookup"><span data-stu-id="61f99-153">This forest is trusted by domains in your on-premises forest.</span></span>

<span data-ttu-id="61f99-154">Ez az architektúra a gyakori felhasználási tartalmaznak objektumok és az identitások, a felhőben tárolt, és a helyszíni egyéni tartományok telepít át a felhő biztonsági elkülönítést karbantartása.</span><span class="sxs-lookup"><span data-stu-id="61f99-154">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="61f99-155">**Előnyei**</span><span class="sxs-lookup"><span data-stu-id="61f99-155">**Benefits**</span></span>

* <span data-ttu-id="61f99-156">A helyszíni identitások megvalósítása, és külön csak Azure identitásokat.</span><span class="sxs-lookup"><span data-stu-id="61f99-156">You can implement on-premises identities and separate Azure-only identities.</span></span>
* <span data-ttu-id="61f99-157">Nem kell replikálni a helyszíni AD-erdőben, az Azure-bA.</span><span class="sxs-lookup"><span data-stu-id="61f99-157">You don't need to replicate from the on-premises AD forest to Azure.</span></span>

<span data-ttu-id="61f99-158">**Kihívásai**</span><span class="sxs-lookup"><span data-stu-id="61f99-158">**Challenges**</span></span>

* <span data-ttu-id="61f99-159">Az Azure a helyszíni identitások hitelesítéshez extra hálózati ugrásokat a helyszíni AD-kiszolgálók.</span><span class="sxs-lookup"><span data-stu-id="61f99-159">Authentication within Azure for on-premises identities requires extra network hops to the on-premises AD servers.</span></span>
* <span data-ttu-id="61f99-160">A saját Active Directory tartományi szolgáltatások kiszolgálót telepíteni, a felhőben erdőben, és erdők közötti bizalmi kapcsolatok létesítése.</span><span class="sxs-lookup"><span data-stu-id="61f99-160">You must deploy your own AD DS servers and forest in the cloud, and establish the appropriate trust relationships between forests.</span></span>

<span data-ttu-id="61f99-161">**[További tudnivalók...][ad-ds-forest]**</span><span class="sxs-lookup"><span data-stu-id="61f99-161">**[Read more...][ad-ds-forest]**</span></span>

## <a name="extend-ad-fs-to-azure"></a><span data-ttu-id="61f99-162">Az Azure Active Directory összevonási szolgáltatások kiterjesztése</span><span class="sxs-lookup"><span data-stu-id="61f99-162">Extend AD FS to Azure</span></span>

<span data-ttu-id="61f99-163">Az Active Directory összevonási szolgáltatások (AD FS) központi telepítése replikálása Azure-ba, az Azure-beli összetevők összevont hitelesítési és engedélyezési végrehajtásához.</span><span class="sxs-lookup"><span data-stu-id="61f99-163">Replicate an Active Directory Federation Services (AD FS) deployment to Azure, to perform federated authentication and authorization for components running in Azure.</span></span> 

<span data-ttu-id="61f99-164">Ez az architektúra a gyakori felhasználási:</span><span class="sxs-lookup"><span data-stu-id="61f99-164">Typical uses for this architecture:</span></span>

* <span data-ttu-id="61f99-165">Felhasználók hitelesítéséhez és engedélyezéséhez a fiókpartner-szervezetek.</span><span class="sxs-lookup"><span data-stu-id="61f99-165">Authenticate and authorize users from partner organizations.</span></span>
* <span data-ttu-id="61f99-166">Lehetővé teszi a felhasználók számára a szervezeti tűzfalon kívül futó webböngészőknek a hitelesítést.</span><span class="sxs-lookup"><span data-stu-id="61f99-166">Allow users to authenticate from web browsers running outside of the organizational firewall.</span></span>
* <span data-ttu-id="61f99-167">Csatlakoztatja a hitelesített külső eszközöket, például a mobileszközök engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="61f99-167">Allow users to connect from authorized external devices such as mobile devices.</span></span> 

<span data-ttu-id="61f99-168">**Előnyei**</span><span class="sxs-lookup"><span data-stu-id="61f99-168">**Benefits**</span></span>

* <span data-ttu-id="61f99-169">Kihasználhatja a jogcímbarát alkalmazásokhoz.</span><span class="sxs-lookup"><span data-stu-id="61f99-169">You can leverage claims-aware applications.</span></span>
* <span data-ttu-id="61f99-170">Lehetővé teszi a hitelesítés külső partnerekkel megbízhatónak.</span><span class="sxs-lookup"><span data-stu-id="61f99-170">Provides the ability to trust external partners for authentication.</span></span>
* <span data-ttu-id="61f99-171">Kompatibilitási nagy számú hitelesítési protokollok megvalósítását végzi.</span><span class="sxs-lookup"><span data-stu-id="61f99-171">Compatibility with large set of authentication protocols.</span></span>

<span data-ttu-id="61f99-172">**Kihívásai**</span><span class="sxs-lookup"><span data-stu-id="61f99-172">**Challenges**</span></span>

* <span data-ttu-id="61f99-173">Ha már telepítette a saját Azure Active Directory tartományi szolgáltatások, az AD FS és az AD FS webalkalmazás-Proxy kiszolgálók.</span><span class="sxs-lookup"><span data-stu-id="61f99-173">You must deploy your own AD DS, AD FS, and AD FS Web Application Proxy servers in Azure.</span></span>
* <span data-ttu-id="61f99-174">Ez az architektúra konfigurálása összetett feladat lehet.</span><span class="sxs-lookup"><span data-stu-id="61f99-174">This architecture can be complex to configure.</span></span>

<span data-ttu-id="61f99-175">**[További tudnivalók...][adfs]**</span><span class="sxs-lookup"><span data-stu-id="61f99-175">**[Read more...][adfs]**</span></span>

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: https://msdn.microsoft.com/library/ms676906.aspx
[adfs]: ./adfs.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
