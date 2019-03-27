---
title: Összevonás az ügyfél AD FS szolgáltatásával
description: Hogyan az ügyfél-val összevont a az AD FS egy több-bérlős alkalmazásban.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: token-cache
pnp.series.next: client-assertion
ms.openlocfilehash: b129d2de8ea3f19df8aa38cc4660885f22f9d9c6
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299638"
---
# <a name="federate-with-a-customers-ad-fs"></a><span data-ttu-id="bf651-103">Összevonás az ügyfél AD FS szolgáltatásával</span><span class="sxs-lookup"><span data-stu-id="bf651-103">Federate with a customer's AD FS</span></span>

<span data-ttu-id="bf651-104">Ez a cikk bemutatja, hogyan egy több-bérlős SaaS-alkalmazáshoz is támogatják a hitelesítést az Active Directory összevonási szolgáltatások (AD FS), annak érdekében, hogy egy ügyfél AD FS vonhat össze.</span><span class="sxs-lookup"><span data-stu-id="bf651-104">This article describes how a multi-tenant SaaS application can support authentication via Active Directory Federation Services (AD FS), in order to federate with a customer's AD FS.</span></span>

## <a name="overview"></a><span data-ttu-id="bf651-105">Áttekintés</span><span class="sxs-lookup"><span data-stu-id="bf651-105">Overview</span></span>

<span data-ttu-id="bf651-106">Az Azure Active Directory (Azure AD) megkönnyíti a felhasználók az Azure AD-bérlő, többek között az Office 365 és Dynamics CRM Online ügyfelek.</span><span class="sxs-lookup"><span data-stu-id="bf651-106">Azure Active Directory (Azure AD) makes it easy to sign in users from Azure AD tenants, including Office365 and Dynamics CRM Online customers.</span></span> <span data-ttu-id="bf651-107">De mi a helyzet ügyfeleink, akik a helyszíni Active Directory a vállalati intraneten?</span><span class="sxs-lookup"><span data-stu-id="bf651-107">But what about customers who use on-premises Active Directory on a corporate intranet?</span></span>

<span data-ttu-id="bf651-108">Az egyik lehetőség van, ezek az ügyfelek számára, hogy szinkronizálja a helyszíni AD és az Azure AD használatával [Azure AD Connect].</span><span class="sxs-lookup"><span data-stu-id="bf651-108">One option is for these customers to sync their on-premises AD with Azure AD, using [Azure AD Connect].</span></span> <span data-ttu-id="bf651-109">Egyes ügyfeleink azonban nem használható ezzel a módszerrel a vállalati informatikai házirend miatt vagy egyéb okból kifolyólag lehet.</span><span class="sxs-lookup"><span data-stu-id="bf651-109">However, some customers may be unable to use this approach, due to corporate IT policy or other reasons.</span></span> <span data-ttu-id="bf651-110">Ebben az esetben egy másik lehetőség, hogy az Active Directory összevonási szolgáltatások (AD FS) keresztül vonhat össze.</span><span class="sxs-lookup"><span data-stu-id="bf651-110">In that case, another option is to federate through Active Directory Federation Services (AD FS).</span></span>

<span data-ttu-id="bf651-111">Ebben a forgatókönyvben engedélyezése:</span><span class="sxs-lookup"><span data-stu-id="bf651-111">To enable this scenario:</span></span>

* <span data-ttu-id="bf651-112">Az ügyfél rendelkezik egy internetkapcsolattal rendelkező AD FS-farm.</span><span class="sxs-lookup"><span data-stu-id="bf651-112">The customer must have an Internet-facing AD FS farm.</span></span>
* <span data-ttu-id="bf651-113">Az SaaS-szolgáltatónak a saját AD FS-farmot helyez üzembe.</span><span class="sxs-lookup"><span data-stu-id="bf651-113">The SaaS provider deploys their own AD FS farm.</span></span>
* <span data-ttu-id="bf651-114">Az ügyfél és az SaaS-szolgáltató be kell állítania a [összevonási megbízhatósági kapcsolatot].</span><span class="sxs-lookup"><span data-stu-id="bf651-114">The customer and the SaaS provider must set up [federation trust].</span></span> <span data-ttu-id="bf651-115">Ez a manuális folyamat.</span><span class="sxs-lookup"><span data-stu-id="bf651-115">This is a manual process.</span></span>

<span data-ttu-id="bf651-116">A megbízhatósági kapcsolatban van a három fő szerepkörök:</span><span class="sxs-lookup"><span data-stu-id="bf651-116">There are three main roles in the trust relation:</span></span>

* <span data-ttu-id="bf651-117">Az ügyfél az AD FS a a [fiókpartner]felelős az ügyfél a felhasználók hitelesítését, ad, és a felhasználói jogcímek biztonsági jogkivonatokat hoz létre.</span><span class="sxs-lookup"><span data-stu-id="bf651-117">The customer's AD FS is the [account partner], responsible for authenticating users from the customer's AD, and creating security tokens with user claims.</span></span>
* <span data-ttu-id="bf651-118">Az SaaS-szolgáltatónak az AD FS a a [erőforráspartner], amely a fiókpartner megbízik, és megkapja a felhasználói jogcímeket.</span><span class="sxs-lookup"><span data-stu-id="bf651-118">The SaaS provider's AD FS is the [resource partner], which trusts the account partner and receives the user claims.</span></span>
* <span data-ttu-id="bf651-119">Az alkalmazás az SaaS-szolgáltatónak az AD FS megbízható függő entitásonkénti (RP) van konfigurálva.</span><span class="sxs-lookup"><span data-stu-id="bf651-119">The application is configured as a relying party (RP) in the SaaS provider's AD FS.</span></span>
  
  ![összevonási megbízhatósági kapcsolat](./images/federation-trust.png)

> [!NOTE]
> <span data-ttu-id="bf651-121">Ez a cikk feltételezzük használja az alkalmazás OpenID Connect hitelesítési protokoll.</span><span class="sxs-lookup"><span data-stu-id="bf651-121">In this article, we assume the application uses OpenID Connect as the authentication protocol.</span></span> <span data-ttu-id="bf651-122">Egy másik lehetőség, hogy a WS-Federation használja.</span><span class="sxs-lookup"><span data-stu-id="bf651-122">Another option is to use WS-Federation.</span></span>
>
> <span data-ttu-id="bf651-123">OpenID Connect az SaaS-szolgáltatónak az AD FS 2016, Windows Server 2016-ban futó kell használnia.</span><span class="sxs-lookup"><span data-stu-id="bf651-123">For OpenID Connect, the SaaS provider must use AD FS 2016, running in Windows Server 2016.</span></span> <span data-ttu-id="bf651-124">Az AD FS 3.0 nem támogatja az OpenID Connect.</span><span class="sxs-lookup"><span data-stu-id="bf651-124">AD FS 3.0 does not support OpenID Connect.</span></span>
>
> <span data-ttu-id="bf651-125">WS-Federation-a-beépített támogatása ASP.NET Core nem tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="bf651-125">ASP.NET Core does not include out-of-the-box support for WS-Federation.</span></span>
>
>

<span data-ttu-id="bf651-126">Az ASP.NET 4 WS-Federation használatának példájáért lásd a [active-directory-dotnet-webapp-wsfederation minta][active-directory-dotnet-webapp-wsfederation].</span><span class="sxs-lookup"><span data-stu-id="bf651-126">For an example of using WS-Federation with ASP.NET 4, see the [active-directory-dotnet-webapp-wsfederation sample][active-directory-dotnet-webapp-wsfederation].</span></span>

## <a name="authentication-flow"></a><span data-ttu-id="bf651-127">A hitelesítési folyamatból</span><span class="sxs-lookup"><span data-stu-id="bf651-127">Authentication flow</span></span>

1. <span data-ttu-id="bf651-128">Amikor a felhasználó a "bejelentkezés" gombra kattint, az alkalmazás átirányítja a SaaS-szolgáltató az AD FS OpenID Connect végpontja.</span><span class="sxs-lookup"><span data-stu-id="bf651-128">When the user clicks "sign in", the application redirects to an OpenID Connect endpoint on the SaaS provider's AD FS.</span></span>
2. <span data-ttu-id="bf651-129">A felhasználó megadja a szervezeti felhasználó nevét ("`alice@corp.contoso.com`").</span><span class="sxs-lookup"><span data-stu-id="bf651-129">The user enters his or her organizational user name ("`alice@corp.contoso.com`").</span></span> <span data-ttu-id="bf651-130">Az AD FS hitelesítőtartomány átirányítása az ügyfél AD FS, ahol a felhasználó megadja a hitelesítő adatokat használja.</span><span class="sxs-lookup"><span data-stu-id="bf651-130">AD FS uses home realm discovery to redirect to the customer's AD FS, where the user enters their credentials.</span></span>
3. <span data-ttu-id="bf651-131">Az ügyfél AD FS elküldi a felhasználói jogcímeket, az SaaS-szolgáltatónak AD FS-ben a WF-összevonás (vagy SAML) használatával.</span><span class="sxs-lookup"><span data-stu-id="bf651-131">The customer's AD FS sends user claims to the SaaS provider's AD FS, using WF-Federation (or SAML).</span></span>
4. <span data-ttu-id="bf651-132">Jogcímek folyamatot az AD FS az alkalmazáshoz, OpenID Connect használatával.</span><span class="sxs-lookup"><span data-stu-id="bf651-132">Claims flow from AD FS to the app, using OpenID Connect.</span></span> <span data-ttu-id="bf651-133">Ehhez a WS-Federation protokollt átállás.</span><span class="sxs-lookup"><span data-stu-id="bf651-133">This requires a protocol transition from WS-Federation.</span></span>

## <a name="limitations"></a><span data-ttu-id="bf651-134">Korlátozások</span><span class="sxs-lookup"><span data-stu-id="bf651-134">Limitations</span></span>

<span data-ttu-id="bf651-135">Alapértelmezés szerint a függő gyártótól származó alkalmazás fogadja az id_token, az alábbi táblázatban látható a rendelkezésre álló jogcímek készletét rögzített.</span><span class="sxs-lookup"><span data-stu-id="bf651-135">By default, the relying party application receives only a fixed set of claims available in the id_token, shown in the following table.</span></span> <span data-ttu-id="bf651-136">Az AD FS 2016 az OpenID Connect forgatókönyvekben id_token szabhatja testre.</span><span class="sxs-lookup"><span data-stu-id="bf651-136">With AD FS 2016, you can customize the id_token in OpenID Connect scenarios.</span></span> <span data-ttu-id="bf651-137">További információkért lásd: [egyéni azonosító-jogkivonatokat az AD FS](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016).</span><span class="sxs-lookup"><span data-stu-id="bf651-137">For more information, see [Custom ID Tokens in AD FS](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016).</span></span>

| <span data-ttu-id="bf651-138">Jogcím</span><span class="sxs-lookup"><span data-stu-id="bf651-138">Claim</span></span> | <span data-ttu-id="bf651-139">Leírás</span><span class="sxs-lookup"><span data-stu-id="bf651-139">Description</span></span> |
| --- | --- |
| <span data-ttu-id="bf651-140">AUD</span><span class="sxs-lookup"><span data-stu-id="bf651-140">aud</span></span> |<span data-ttu-id="bf651-141">Célközönség.</span><span class="sxs-lookup"><span data-stu-id="bf651-141">Audience.</span></span> <span data-ttu-id="bf651-142">Az alkalmazás, amelynek a jogcímek adtak ki.</span><span class="sxs-lookup"><span data-stu-id="bf651-142">The application for which the claims were issued.</span></span> |
| <span data-ttu-id="bf651-143">AuthenticationInstant</span><span class="sxs-lookup"><span data-stu-id="bf651-143">authenticationinstant</span></span> |<span data-ttu-id="bf651-144">[Azonnali hitelesítés].</span><span class="sxs-lookup"><span data-stu-id="bf651-144">[Authentication instant].</span></span> <span data-ttu-id="bf651-145">Melyik hitelesítési ideje történt.</span><span class="sxs-lookup"><span data-stu-id="bf651-145">The time at which authentication occurred.</span></span> |
| <span data-ttu-id="bf651-146">c_hash</span><span class="sxs-lookup"><span data-stu-id="bf651-146">c_hash</span></span> |<span data-ttu-id="bf651-147">Kód kivonatértéket.</span><span class="sxs-lookup"><span data-stu-id="bf651-147">Code hash value.</span></span> <span data-ttu-id="bf651-148">Ez a jogkivonat tartalma kivonatát.</span><span class="sxs-lookup"><span data-stu-id="bf651-148">This is a hash of the token contents.</span></span> |
| <span data-ttu-id="bf651-149">Exp</span><span class="sxs-lookup"><span data-stu-id="bf651-149">exp</span></span> |<span data-ttu-id="bf651-150">[Lejárati idő].</span><span class="sxs-lookup"><span data-stu-id="bf651-150">[Expiration time].</span></span> <span data-ttu-id="bf651-151">Az idő, amely után a jogkivonatot már nem fogadható el.</span><span class="sxs-lookup"><span data-stu-id="bf651-151">The time after which the token will no longer be accepted.</span></span> |
| <span data-ttu-id="bf651-152">IAT</span><span class="sxs-lookup"><span data-stu-id="bf651-152">iat</span></span> |<span data-ttu-id="bf651-153">Kiállított.</span><span class="sxs-lookup"><span data-stu-id="bf651-153">Issued at.</span></span> <span data-ttu-id="bf651-154">Az az időpont, amikor a jogkivonat lett kiállítva.</span><span class="sxs-lookup"><span data-stu-id="bf651-154">The time when the token was issued.</span></span> |
| <span data-ttu-id="bf651-155">iss</span><span class="sxs-lookup"><span data-stu-id="bf651-155">iss</span></span> |<span data-ttu-id="bf651-156">Kibocsátó.</span><span class="sxs-lookup"><span data-stu-id="bf651-156">Issuer.</span></span> <span data-ttu-id="bf651-157">Ez a jogcím értéke mindig az erőforráspartner az AD FS.</span><span class="sxs-lookup"><span data-stu-id="bf651-157">The value of this claim is always the resource partner's AD FS.</span></span> |
| <span data-ttu-id="bf651-158">név</span><span class="sxs-lookup"><span data-stu-id="bf651-158">name</span></span> |<span data-ttu-id="bf651-159">A felhasználó neve.</span><span class="sxs-lookup"><span data-stu-id="bf651-159">User name.</span></span> <span data-ttu-id="bf651-160">Például: `john@corp.fabrikam.com`</span><span class="sxs-lookup"><span data-stu-id="bf651-160">Example: `john@corp.fabrikam.com`</span></span> |
| <span data-ttu-id="bf651-161">nameidentifier</span><span class="sxs-lookup"><span data-stu-id="bf651-161">nameidentifier</span></span> |<span data-ttu-id="bf651-162">[Névazonosító].</span><span class="sxs-lookup"><span data-stu-id="bf651-162">[Name identifier].</span></span> <span data-ttu-id="bf651-163">A neve, amelyhez a jogkivonatot adta ki az entitás azonosítója.</span><span class="sxs-lookup"><span data-stu-id="bf651-163">The identifier for the name of the entity for which the token was issued.</span></span> |
| <span data-ttu-id="bf651-164">egyszeri</span><span class="sxs-lookup"><span data-stu-id="bf651-164">nonce</span></span> |<span data-ttu-id="bf651-165">Munkamenet egyszeri.</span><span class="sxs-lookup"><span data-stu-id="bf651-165">Session nonce.</span></span> <span data-ttu-id="bf651-166">Az AD FS-ismétléses támadások megelőzése érdekében által generált egyedi érték.</span><span class="sxs-lookup"><span data-stu-id="bf651-166">A unique value generated by AD FS to help prevent replay attacks.</span></span> |
| <span data-ttu-id="bf651-167">egyszerű felhasználónév</span><span class="sxs-lookup"><span data-stu-id="bf651-167">upn</span></span> |<span data-ttu-id="bf651-168">Egyszerű felhasználónév (UPN).</span><span class="sxs-lookup"><span data-stu-id="bf651-168">User principal name (UPN).</span></span> <span data-ttu-id="bf651-169">Például: `john@corp.fabrikam.com`</span><span class="sxs-lookup"><span data-stu-id="bf651-169">Example: `john@corp.fabrikam.com`</span></span> |
| <span data-ttu-id="bf651-170">pwd_exp</span><span class="sxs-lookup"><span data-stu-id="bf651-170">pwd_exp</span></span> |<span data-ttu-id="bf651-171">Jelszó lejárati idejét.</span><span class="sxs-lookup"><span data-stu-id="bf651-171">Password expiration period.</span></span> <span data-ttu-id="bf651-172">Amíg a felhasználó jelszavát vagy egy hasonló hitelesítés titkos, például a PIN-kód másodpercek számát.</span><span class="sxs-lookup"><span data-stu-id="bf651-172">The number of seconds until the user's password or a similar authentication secret, such as a PIN.</span></span> <span data-ttu-id="bf651-173">lejár.</span><span class="sxs-lookup"><span data-stu-id="bf651-173">expires.</span></span> |

> [!NOTE]
> <span data-ttu-id="bf651-174">A "iss" jogcím tartalmazza az AD FS, a partner (általában ez a jogcím azonosítja a SaaS-szolgáltató a kibocsátó).</span><span class="sxs-lookup"><span data-stu-id="bf651-174">The "iss" claim contains the AD FS of the partner (typically, this claim will identify the SaaS provider as the issuer).</span></span> <span data-ttu-id="bf651-175">Azt határozza meg az ügyfél AD FS.</span><span class="sxs-lookup"><span data-stu-id="bf651-175">It does not identify the customer's AD FS.</span></span> <span data-ttu-id="bf651-176">Az egyszerű felhasználónév részeként az ügyfél tartományban található.</span><span class="sxs-lookup"><span data-stu-id="bf651-176">You can find the customer's domain as part of the UPN.</span></span>

<span data-ttu-id="bf651-177">Ez a cikk ismerteti, hogyan lehet az RP-ből (az alkalmazás) és a fiókpartner (az ügyfél) közötti bizalmi kapcsolat beállítása.</span><span class="sxs-lookup"><span data-stu-id="bf651-177">The rest of this article describes how to set up the trust relationship between the RP (the app) and the account partner (the customer).</span></span>

## <a name="ad-fs-deployment"></a><span data-ttu-id="bf651-178">Az AD FS üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="bf651-178">AD FS deployment</span></span>

<span data-ttu-id="bf651-179">Az SaaS-szolgáltató telepítheti az AD FS helyszíni vagy Azure virtuális gépeken.</span><span class="sxs-lookup"><span data-stu-id="bf651-179">The SaaS provider can deploy AD FS either on-premises or on Azure VMs.</span></span> <span data-ttu-id="bf651-180">A biztonsági és rendelkezésre állás a következő irányelveket fontosak:</span><span class="sxs-lookup"><span data-stu-id="bf651-180">For security and availability, the following guidelines are important:</span></span>

* <span data-ttu-id="bf651-181">Legalább két AD FS-kiszolgálók és a két az AD FS szolgáltatás ajánlott rendelkezésre állásának eléréséhez az AD FS-proxykiszolgáló üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="bf651-181">Deploy at least two AD FS servers and two AD FS proxy servers to achieve the best availability of the AD FS service.</span></span>
* <span data-ttu-id="bf651-182">A tartományvezérlők és AD FS-kiszolgálók kell soha nem szabad közvetlenül elérhetővé tenni az interneten, és a közvetlen hozzáférést a azokat egy virtuális hálózatban kell lennie.</span><span class="sxs-lookup"><span data-stu-id="bf651-182">Domain controllers and AD FS servers should never be exposed directly to the Internet and should be in a virtual network with direct access to them.</span></span>
* <span data-ttu-id="bf651-183">Webalkalmazás-proxyt (korábban az AD FS proxy) közzététele AD FS-kiszolgálók internetkapcsolattal kell használni.</span><span class="sxs-lookup"><span data-stu-id="bf651-183">Web application proxies (previously AD FS proxies) must be used to publish AD FS servers to the Internet.</span></span>

<span data-ttu-id="bf651-184">Állítsa be a hasonló topológia az Azure virtuális hálózatok, NSG, az azure virtuális gép és a rendelkezésre állási csoportok használatát igényli.</span><span class="sxs-lookup"><span data-stu-id="bf651-184">To set up a similar topology in Azure requires the use of Virtual networks, NSG’s, azure VM’s and availability sets.</span></span> <span data-ttu-id="bf651-185">További részletekért lásd: [központi telepítése Windows Server Active Directory az Azure Virtual Machinesben irányelvek][active-directory-on-azure].</span><span class="sxs-lookup"><span data-stu-id="bf651-185">For more details, see [Guidelines for Deploying Windows Server Active Directory on Azure Virtual Machines][active-directory-on-azure].</span></span>

## <a name="configure-openid-connect-authentication-with-ad-fs"></a><span data-ttu-id="bf651-186">OpenID Connect-hitelesítés konfigurálása az AD FS-sel</span><span class="sxs-lookup"><span data-stu-id="bf651-186">Configure OpenID Connect authentication with AD FS</span></span>

<span data-ttu-id="bf651-187">Az SaaS-szolgáltató engedélyeznie kell a OpenID Connect az alkalmazások és az Active Directory összevonási szolgáltatások között.</span><span class="sxs-lookup"><span data-stu-id="bf651-187">The SaaS provider must enable OpenID Connect between the application and AD FS.</span></span> <span data-ttu-id="bf651-188">Ehhez adja hozzá az AD FS-ben egy csoportot.</span><span class="sxs-lookup"><span data-stu-id="bf651-188">To do so, add an application group in AD FS.</span></span>  <span data-ttu-id="bf651-189">Ezen részletes utasítások találhatók [blogbejegyzés], a "Webes alkalmazás beállítása az OpenId Connect jelentkezzen be az AD FS."</span><span class="sxs-lookup"><span data-stu-id="bf651-189">You can find detailed instructions in this [blog post], under " Setting up a Web App for OpenId Connect sign in AD FS."</span></span>

<span data-ttu-id="bf651-190">Ezután konfigurálja az OpenID Connect közbenső szoftvert.</span><span class="sxs-lookup"><span data-stu-id="bf651-190">Next, configure the OpenID Connect middleware.</span></span> <span data-ttu-id="bf651-191">A metaadatok végpontja `https://domain/adfs/.well-known/openid-configuration`, ahol a tartomány az SaaS-szolgáltatónak az AD FS-tartomány.</span><span class="sxs-lookup"><span data-stu-id="bf651-191">The metadata endpoint is `https://domain/adfs/.well-known/openid-configuration`, where domain is the SaaS provider's AD FS domain.</span></span>

<span data-ttu-id="bf651-192">Általában akkor lehet, hogy kombinálva ez más OpenID Connect végpontok (például az AAD-).</span><span class="sxs-lookup"><span data-stu-id="bf651-192">Typically you might combine this with other OpenID Connect endpoints (such as AAD).</span></span> <span data-ttu-id="bf651-193">Két különböző bejelentkezési gombot vagy más módon megkülönböztetésükhöz, úgy, hogy a felhasználó a megfelelő hitelesítési végpont küldendő kell.</span><span class="sxs-lookup"><span data-stu-id="bf651-193">You'll need two different sign-in buttons or some other way to distinguish them, so that the user is sent to the correct authentication endpoint.</span></span>

## <a name="configure-the-ad-fs-resource-partner"></a><span data-ttu-id="bf651-194">Az AD FS erőforráspartner konfigurálása</span><span class="sxs-lookup"><span data-stu-id="bf651-194">Configure the AD FS Resource Partner</span></span>

<span data-ttu-id="bf651-195">Az SaaS-szolgáltató minden egyes ügyfél, amely szeretne az ADFS-n keresztül csatlakozik a következőket kell tennie:</span><span class="sxs-lookup"><span data-stu-id="bf651-195">The SaaS provider must do the following for each customer that wants to connect via ADFS:</span></span>

1. <span data-ttu-id="bf651-196">Jogcímszolgáltatói megbízhatóság hozzáadása.</span><span class="sxs-lookup"><span data-stu-id="bf651-196">Add a claims provider trust.</span></span>
2. <span data-ttu-id="bf651-197">Adja hozzá a jogcímszabályokat.</span><span class="sxs-lookup"><span data-stu-id="bf651-197">Add claims rules.</span></span>
3. <span data-ttu-id="bf651-198">A kezdőtartomány felderítés engedélyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="bf651-198">Enable home-realm discovery.</span></span>

<span data-ttu-id="bf651-199">További részleteket az alábbiakban a lépéseket.</span><span class="sxs-lookup"><span data-stu-id="bf651-199">Here are the steps in more detail.</span></span>

### <a name="add-the-claims-provider-trust"></a><span data-ttu-id="bf651-200">A jogcím-szolgáltatói megbízhatóság hozzáadása</span><span class="sxs-lookup"><span data-stu-id="bf651-200">Add the claims provider trust</span></span>

1. <span data-ttu-id="bf651-201">A Kiszolgálókezelőben kattintson a **eszközök**, majd válassza ki **AD FS-kezelőben**.</span><span class="sxs-lookup"><span data-stu-id="bf651-201">In Server Manager, click **Tools**, and then select **AD FS Management**.</span></span>
2. <span data-ttu-id="bf651-202">A konzolfán a **az AD FS**, kattintson a jobb gombbal **jogcím-szolgáltatói Megbízhatóságok**.</span><span class="sxs-lookup"><span data-stu-id="bf651-202">In the console tree, under **AD FS**, right click **Claims Provider Trusts**.</span></span> <span data-ttu-id="bf651-203">Válassza ki **jogcím-szolgáltatói megbízhatóság hozzáadása**.</span><span class="sxs-lookup"><span data-stu-id="bf651-203">Select **Add Claims Provider Trust**.</span></span>
3. <span data-ttu-id="bf651-204">Kattintson a **Start** varázsló elindításához.</span><span class="sxs-lookup"><span data-stu-id="bf651-204">Click **Start** to start the wizard.</span></span>
4. <span data-ttu-id="bf651-205">Válassza ki az beállítás "importálási adatok a jogcímszolgáltató, illetve online vagy helyi hálózaton közzétett kapcsolatban".</span><span class="sxs-lookup"><span data-stu-id="bf651-205">Select the option "Import data about the claims provider published online or on a local network".</span></span> <span data-ttu-id="bf651-206">Adja meg az ügyfél összevonási metaadatok végpontjának URI.</span><span class="sxs-lookup"><span data-stu-id="bf651-206">Enter the URI of the customer's federation metadata endpoint.</span></span> <span data-ttu-id="bf651-207">(Példa: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`.) Szüksége lesz a probléma az ügyféltől.</span><span class="sxs-lookup"><span data-stu-id="bf651-207">(Example: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`.) You will need to get this from the customer.</span></span>
5. <span data-ttu-id="bf651-208">Fejezze be a varázslót, az alapértelmezett beállításokat használva.</span><span class="sxs-lookup"><span data-stu-id="bf651-208">Complete the wizard using the default options.</span></span>

### <a name="edit-claims-rules"></a><span data-ttu-id="bf651-209">Jogcímszabályok szerkesztése</span><span class="sxs-lookup"><span data-stu-id="bf651-209">Edit claims rules</span></span>

1. <span data-ttu-id="bf651-210">Kattintson a jobb gombbal az újonnan hozzáadott jogcím-szolgáltatói megbízhatóság, és válassza ki **Jogcímszabályok szerkesztése**.</span><span class="sxs-lookup"><span data-stu-id="bf651-210">Right-click the newly added claims provider trust, and select **Edit Claims Rules**.</span></span>
2. <span data-ttu-id="bf651-211">Kattintson a **szabály hozzáadása**.</span><span class="sxs-lookup"><span data-stu-id="bf651-211">Click **Add Rule**.</span></span>
3. <span data-ttu-id="bf651-212">Válassza ki a "Haladnak keresztül vagy a szűrő egy bejövő jogcím", és kattintson a **tovább**.</span><span class="sxs-lookup"><span data-stu-id="bf651-212">Select "Pass Through or Filter an Incoming Claim" and click **Next**.</span></span>
   <span data-ttu-id="bf651-213">![Átalakítási jogcímszabály hozzáadása varázsló](./images/edit-claims-rule.png)</span><span class="sxs-lookup"><span data-stu-id="bf651-213">![Add Transform Claim Rule Wizard](./images/edit-claims-rule.png)</span></span>
4. <span data-ttu-id="bf651-214">Adja meg a szabály nevét.</span><span class="sxs-lookup"><span data-stu-id="bf651-214">Enter a name for the rule.</span></span>
5. <span data-ttu-id="bf651-215">Válassza a "Bejövő jogcím típusa" **UPN**.</span><span class="sxs-lookup"><span data-stu-id="bf651-215">Under "Incoming claim type", select **UPN**.</span></span>
6. <span data-ttu-id="bf651-216">Válassza ki a "Az összes jogcímérték továbbítása".</span><span class="sxs-lookup"><span data-stu-id="bf651-216">Select "Pass through all claim values".</span></span>
   <span data-ttu-id="bf651-217">![Átalakítási jogcímszabály hozzáadása varázsló](./images/edit-claims-rule2.png)</span><span class="sxs-lookup"><span data-stu-id="bf651-217">![Add Transform Claim Rule Wizard](./images/edit-claims-rule2.png)</span></span>
7. <span data-ttu-id="bf651-218">Kattintson a **Befejezés** gombra.</span><span class="sxs-lookup"><span data-stu-id="bf651-218">Click **Finish**.</span></span>
8. <span data-ttu-id="bf651-219">Ismételje meg a 2 – 7, és adja meg **Forráshorgony jogcím típusa** számára a bejövő jogcím típusa.</span><span class="sxs-lookup"><span data-stu-id="bf651-219">Repeat steps 2 - 7, and specify **Anchor Claim Type** for the incoming claim type.</span></span>
9. <span data-ttu-id="bf651-220">Kattintson a **OK** a varázsló befejezéséhez.</span><span class="sxs-lookup"><span data-stu-id="bf651-220">Click **OK** to complete the wizard.</span></span>

### <a name="enable-home-realm-discovery"></a><span data-ttu-id="bf651-221">A kezdőtartomány felderítés engedélyezéséhez</span><span class="sxs-lookup"><span data-stu-id="bf651-221">Enable home-realm discovery</span></span>

<span data-ttu-id="bf651-222">Futtassa a következő PowerShell-parancsfájlt:</span><span class="sxs-lookup"><span data-stu-id="bf651-222">Run the following PowerShell script:</span></span>

```powershell
Set-ADFSClaimsProviderTrust -TargetName "name" -OrganizationalAccountSuffix @("suffix")
```

<span data-ttu-id="bf651-223">Ha "name" a valódi neve az a jogcím-szolgáltatói megbízhatóság, és "utótag" az ügyfél az UPN-utótagot a AD, (például "corp.fabrikam.com").</span><span class="sxs-lookup"><span data-stu-id="bf651-223">where "name" is the friendly name of the claims provider trust, and "suffix" is the UPN suffix for the customer's AD (example, "corp.fabrikam.com").</span></span>

<span data-ttu-id="bf651-224">Ezzel a konfigurációval a végfelhasználók írja be a saját szervezeti fiókjukba, és az AD FS automatikusan kiválasztja a megfelelő jogcímszolgáltatót.</span><span class="sxs-lookup"><span data-stu-id="bf651-224">With this configuration, end users can type in their organizational account, and AD FS automatically selects the corresponding claims provider.</span></span> <span data-ttu-id="bf651-225">Lásd: [az AD FS bejelentkezési oldalainak testreszabása], "Configure identitásszolgáltató bizonyos e-mail utótagok használata" területen.</span><span class="sxs-lookup"><span data-stu-id="bf651-225">See [Customizing the AD FS Sign-in Pages], under the section "Configure Identity Provider to use certain email suffixes".</span></span>

## <a name="configure-the-ad-fs-account-partner"></a><span data-ttu-id="bf651-226">Az AD FS-fiók partnerének beállítása</span><span class="sxs-lookup"><span data-stu-id="bf651-226">Configure the AD FS Account Partner</span></span>

<span data-ttu-id="bf651-227">Az ügyfél a következőket kell tennie:</span><span class="sxs-lookup"><span data-stu-id="bf651-227">The customer must do the following:</span></span>

1. <span data-ttu-id="bf651-228">Adjon hozzá egy függő entitásonkénti (RP) megbízhatóságát.</span><span class="sxs-lookup"><span data-stu-id="bf651-228">Add a relying party (RP) trust.</span></span>
2. <span data-ttu-id="bf651-229">Hozzáadja a jogcímszabályokat.</span><span class="sxs-lookup"><span data-stu-id="bf651-229">Adds claims rules.</span></span>

### <a name="add-the-rp-trust"></a><span data-ttu-id="bf651-230">A függő Entitás megbízhatóságának hozzáadása</span><span class="sxs-lookup"><span data-stu-id="bf651-230">Add the RP trust</span></span>

1. <span data-ttu-id="bf651-231">A Kiszolgálókezelőben kattintson a **eszközök**, majd válassza ki **AD FS-kezelőben**.</span><span class="sxs-lookup"><span data-stu-id="bf651-231">In Server Manager, click **Tools**, and then select **AD FS Management**.</span></span>
2. <span data-ttu-id="bf651-232">A konzolfán a **az AD FS**, kattintson a jobb gombbal **függő entitás Megbízhatóságai**.</span><span class="sxs-lookup"><span data-stu-id="bf651-232">In the console tree, under **AD FS**, right click **Relying Party Trusts**.</span></span> <span data-ttu-id="bf651-233">Válassza ki **függő entitás megbízhatóságának hozzáadása**.</span><span class="sxs-lookup"><span data-stu-id="bf651-233">Select **Add Relying Party Trust**.</span></span>
3. <span data-ttu-id="bf651-234">Válassza ki **jogcímeket figyelembe** kattintson **Start**.</span><span class="sxs-lookup"><span data-stu-id="bf651-234">Select **Claims Aware** and click **Start**.</span></span>
4. <span data-ttu-id="bf651-235">Az a **adatforrás kiválasztása** lapra, jelölje be a "Importálási adatok a jogcímszolgáltató közzé online vagy helyi hálózaton" lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="bf651-235">On the **Select Data Source** page, select the option "Import data about the claims provider published online or on a local network".</span></span> <span data-ttu-id="bf651-236">Adja meg a SaaS-szolgáltató összevonási metaadatok végpontjának URI.</span><span class="sxs-lookup"><span data-stu-id="bf651-236">Enter the URI of the SaaS provider's federation metadata endpoint.</span></span>
   <span data-ttu-id="bf651-237">![Adja hozzá a függő entitás megbízhatóságának varázsló](./images/add-rp-trust.png)</span><span class="sxs-lookup"><span data-stu-id="bf651-237">![Add Relying Party Trust Wizard](./images/add-rp-trust.png)</span></span>
5. <span data-ttu-id="bf651-238">Az a **megjelenítendő név megadása** lapon, bármilyen nevet adja.</span><span class="sxs-lookup"><span data-stu-id="bf651-238">On the **Specify Display Name** page, enter any name.</span></span>
6. <span data-ttu-id="bf651-239">Az a **válassza a hozzáférés-vezérlési házirend** lapon, válassza ki a szabályzatot.</span><span class="sxs-lookup"><span data-stu-id="bf651-239">On the **Choose Access Control Policy** page, choose a policy.</span></span> <span data-ttu-id="bf651-240">A szervezet engedélyezése mindenki számára, vagy egy adott biztonsági csoport kiválasztása.</span><span class="sxs-lookup"><span data-stu-id="bf651-240">You could permit everyone in the organization, or choose a specific security group.</span></span>
   <span data-ttu-id="bf651-241">![Adja hozzá a függő entitás megbízhatóságának varázsló](./images/add-rp-trust2.png)</span><span class="sxs-lookup"><span data-stu-id="bf651-241">![Add Relying Party Trust Wizard](./images/add-rp-trust2.png)</span></span>
7. <span data-ttu-id="bf651-242">Adja meg a szükséges paramétereket a **házirend** mezőbe.</span><span class="sxs-lookup"><span data-stu-id="bf651-242">Enter any parameters required in the **Policy** box.</span></span>
8. <span data-ttu-id="bf651-243">Kattintson a **tovább** a varázsló befejezéséhez.</span><span class="sxs-lookup"><span data-stu-id="bf651-243">Click **Next** to complete the wizard.</span></span>

### <a name="add-claims-rules"></a><span data-ttu-id="bf651-244">A szabályok hozzáadása</span><span class="sxs-lookup"><span data-stu-id="bf651-244">Add claims rules</span></span>

1. <span data-ttu-id="bf651-245">Kattintson a jobb gombbal az újonnan hozzáadott függőentitás-megbízhatóságot, és válassza ki **jogcím-kiállítási szabályzat szerkesztése**.</span><span class="sxs-lookup"><span data-stu-id="bf651-245">Right-click the newly added relying party trust, and select **Edit Claim Issuance Policy**.</span></span>
2. <span data-ttu-id="bf651-246">Kattintson a **szabály hozzáadása**.</span><span class="sxs-lookup"><span data-stu-id="bf651-246">Click **Add Rule**.</span></span>
3. <span data-ttu-id="bf651-247">Válassza ki a "Küldés LDAP attribútumok szerint jogcímek", és kattintson a **tovább**.</span><span class="sxs-lookup"><span data-stu-id="bf651-247">Select "Send LDAP Attributes as Claims" and click **Next**.</span></span>
4. <span data-ttu-id="bf651-248">Adjon meg egy nevet a szabálynak, például az "Egyszerű".</span><span class="sxs-lookup"><span data-stu-id="bf651-248">Enter a name for the rule, such as "UPN".</span></span>
5. <span data-ttu-id="bf651-249">A **attribútumtár**válassza **Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="bf651-249">Under **Attribute store**, select **Active Directory**.</span></span>
   <span data-ttu-id="bf651-250">![Átalakítási jogcímszabály hozzáadása varázsló](./images/add-claims-rules.png)</span><span class="sxs-lookup"><span data-stu-id="bf651-250">![Add Transform Claim Rule Wizard](./images/add-claims-rules.png)</span></span>
6. <span data-ttu-id="bf651-251">Az a **LDAP attribútumok leképezése** szakaszban:</span><span class="sxs-lookup"><span data-stu-id="bf651-251">In the **Mapping of LDAP attributes** section:</span></span>
   * <span data-ttu-id="bf651-252">A **LDAP attribútum**válassza **felhasználónév-egyszerű**.</span><span class="sxs-lookup"><span data-stu-id="bf651-252">Under **LDAP Attribute**, select **User-Principal-Name**.</span></span>
   * <span data-ttu-id="bf651-253">A **kimenő jogcímtípus**válassza **UPN**.</span><span class="sxs-lookup"><span data-stu-id="bf651-253">Under **Outgoing Claim Type**, select **UPN**.</span></span>
     <span data-ttu-id="bf651-254">![Átalakítási jogcímszabály hozzáadása varázsló](./images/add-claims-rules2.png)</span><span class="sxs-lookup"><span data-stu-id="bf651-254">![Add Transform Claim Rule Wizard](./images/add-claims-rules2.png)</span></span>
7. <span data-ttu-id="bf651-255">Kattintson a **Befejezés** gombra.</span><span class="sxs-lookup"><span data-stu-id="bf651-255">Click **Finish**.</span></span>
8. <span data-ttu-id="bf651-256">Kattintson a **szabály hozzáadása** újra.</span><span class="sxs-lookup"><span data-stu-id="bf651-256">Click **Add Rule** again.</span></span>
9. <span data-ttu-id="bf651-257">Válassza ki a "Küldés jogcímek használata egy egyéni szabály", és kattintson a **tovább**.</span><span class="sxs-lookup"><span data-stu-id="bf651-257">Select "Send Claims Using a Custom Rule" and click **Next**.</span></span>
10. <span data-ttu-id="bf651-258">Adjon meg egy nevet a szabálynak, például a "Forráshorgony jogcímtípus".</span><span class="sxs-lookup"><span data-stu-id="bf651-258">Enter a name for the rule, such as "Anchor Claim Type".</span></span>
11. <span data-ttu-id="bf651-259">A **egyéni szabály**, írja be a következőket:</span><span class="sxs-lookup"><span data-stu-id="bf651-259">Under **Custom rule**, enter the following:</span></span>

    ```console
    EXISTS([Type == "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype"])=>
    issue (Type = "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype",
          Value = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn");
    ```

    <span data-ttu-id="bf651-260">Ez a szabály kiad egy jogcímet típusú `anchorclaimtype`.</span><span class="sxs-lookup"><span data-stu-id="bf651-260">This rule issues a claim of type `anchorclaimtype`.</span></span> <span data-ttu-id="bf651-261">A jogcím arra utasítja a függő entitás használandó egyszerű felhasználónév a felhasználó azonosítója nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="bf651-261">The claim tells the relying party to use UPN as the user's immutable ID.</span></span>
12. <span data-ttu-id="bf651-262">Kattintson a **Befejezés** gombra.</span><span class="sxs-lookup"><span data-stu-id="bf651-262">Click **Finish**.</span></span>
13. <span data-ttu-id="bf651-263">Kattintson a **OK** a varázsló befejezéséhez.</span><span class="sxs-lookup"><span data-stu-id="bf651-263">Click **OK** to complete the wizard.</span></span>

<!-- links -->

[Azure AD Connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
[összevonási megbízhatósági kapcsolatot]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[federation trust]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[fiókpartner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[account partner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[erőforráspartner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[resource partner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[Azonnali hitelesítés]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[Authentication instant]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[Lejárati idő]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[Expiration time]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[Névazonosító]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[Name identifier]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[active-directory-on-azure]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[Blogbejegyzés]: https://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[blog post]: https://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[Az AD FS bejelentkezési oldalainak testreszabása]: https://technet.microsoft.com/library/dn280950.aspx
[Customizing the AD FS Sign-in Pages]: https://technet.microsoft.com/library/dn280950.aspx
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[client assertion]: client-assertion.md
[active-directory-dotnet-webapp-wsfederation]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation
