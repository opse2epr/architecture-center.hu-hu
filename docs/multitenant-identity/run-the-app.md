---
title: A Surveys alkalmazás futtatása
description: A felmérések mintaalkalmazás futtatása helyben
author: MikeWasson
ms:date: 07/21/2017
ms.openlocfilehash: 28d976374e5d6dbad434873eef149704f26a1f3f
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="run-the-surveys-application"></a><span data-ttu-id="9b04d-103">A Surveys alkalmazás futtatása</span><span class="sxs-lookup"><span data-stu-id="9b04d-103">Run the Surveys application</span></span>

<span data-ttu-id="9b04d-104">Ez a cikk ismerteti, hogyan futtathatók a [Dejójáték felmérések](./tailspin.md) helyileg, a Visual Studio alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="9b04d-104">This article describes how to run the [Tailspin Surveys](./tailspin.md) application locally, from Visual Studio.</span></span> <span data-ttu-id="9b04d-105">Ezeket a lépéseket az Azure alkalmazás nem telepít.</span><span class="sxs-lookup"><span data-stu-id="9b04d-105">In these steps, you won't deploy the application to Azure.</span></span> <span data-ttu-id="9b04d-106">Azonban szüksége lesz egy Azure-erőforrások létrehozásához &mdash; az Azure Active Directory (Azure AD) címtár és a Redis gyorsítótár.</span><span class="sxs-lookup"><span data-stu-id="9b04d-106">However, you will need to create some Azure resources &mdash; an Azure Active Directory (Azure AD) directory and a Redis cache.</span></span>

<span data-ttu-id="9b04d-107">A lépések összefoglalása itt található:</span><span class="sxs-lookup"><span data-stu-id="9b04d-107">Here is a summary of the steps:</span></span>

1. <span data-ttu-id="9b04d-108">Hozzon létre egy Azure AD-címtár (bérlői) Dejójáték fiktív cég.</span><span class="sxs-lookup"><span data-stu-id="9b04d-108">Create an Azure AD directory (tenant) for the fictitious Tailspin company.</span></span>
2. <span data-ttu-id="9b04d-109">Regisztrálja a felmérések alkalmazás és a háttérkiszolgáló webes API-t az Azure AD.</span><span class="sxs-lookup"><span data-stu-id="9b04d-109">Register the Surveys application and the backend web API with Azure AD.</span></span>
3. <span data-ttu-id="9b04d-110">Azure Redis Cache példányt létrehozni.</span><span class="sxs-lookup"><span data-stu-id="9b04d-110">Create an Azure Redis Cache instance.</span></span>
4. <span data-ttu-id="9b04d-111">Az Alkalmazásbeállítások konfigurálása, és hozzon létre egy helyi adatbázist.</span><span class="sxs-lookup"><span data-stu-id="9b04d-111">Configure application settings and create a local database.</span></span>
5. <span data-ttu-id="9b04d-112">Futtassa az alkalmazást, és regisztráljon egy új bérlőt.</span><span class="sxs-lookup"><span data-stu-id="9b04d-112">Run the application and sign up a new tenant.</span></span>
6. <span data-ttu-id="9b04d-113">Alkalmazás-szerepkörök hozzáadása a felhasználók számára.</span><span class="sxs-lookup"><span data-stu-id="9b04d-113">Add application roles to users.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="9b04d-114">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="9b04d-114">Prerequisites</span></span>
-   <span data-ttu-id="9b04d-115">[Visual Studio 2017][VS2017]</span><span class="sxs-lookup"><span data-stu-id="9b04d-115">[Visual Studio 2017][VS2017]</span></span>
-   <span data-ttu-id="9b04d-116">[A Microsoft Azure](https://azure.microsoft.com) fiók</span><span class="sxs-lookup"><span data-stu-id="9b04d-116">[Microsoft Azure](https://azure.microsoft.com) account</span></span>

## <a name="create-the-tailspin-tenant"></a><span data-ttu-id="9b04d-117">A Dejójáték bérlő létrehozása</span><span class="sxs-lookup"><span data-stu-id="9b04d-117">Create the Tailspin tenant</span></span>

<span data-ttu-id="9b04d-118">Dejójáték fiktív cég, amelyen a felmérések alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="9b04d-118">Tailspin is the fictitious company that hosts the Surveys application.</span></span> <span data-ttu-id="9b04d-119">Dejójáték használja az Azure AD és az alkalmazás regisztrálásához más bérlők engedélyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="9b04d-119">Tailspin uses Azure AD to enable other tenants to register with the app.</span></span> <span data-ttu-id="9b04d-120">Ezek az ügyfelek használhatja az Azure AD hitelesítő adatait az alkalmazás-ba való bejelentkezéshez.</span><span class="sxs-lookup"><span data-stu-id="9b04d-120">Those customers can then use their Azure AD credentials to sign into the app.</span></span>

<span data-ttu-id="9b04d-121">Ebben a lépésben létre fog hozni egy Azure AD-címtár Dejójáték számára.</span><span class="sxs-lookup"><span data-stu-id="9b04d-121">In this step, you'll create an Azure AD directory for Tailspin.</span></span>

1. <span data-ttu-id="9b04d-122">Jelentkezzen be az [Azure Portalra][portal].</span><span class="sxs-lookup"><span data-stu-id="9b04d-122">Sign into the [Azure portal][portal].</span></span>

2. <span data-ttu-id="9b04d-123">Kattintson a **új** > **biztonság + identitás szakaszában** > **az Azure Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-123">Click **New** > **Security + Identity** > **Azure Active Directory**.</span></span>

3. <span data-ttu-id="9b04d-124">Adja meg `Tailspin` a szervezet neve, és adja meg a tartomány nevét.</span><span class="sxs-lookup"><span data-stu-id="9b04d-124">Enter `Tailspin` for the organization name, and enter a domain name.</span></span> <span data-ttu-id="9b04d-125">A tartománynév fog rendelkezni az űrlap `xxxx.onmicrosoft.com` és egyedinek kell lenniük.</span><span class="sxs-lookup"><span data-stu-id="9b04d-125">The domain name will have the form `xxxx.onmicrosoft.com` and must be globally unique.</span></span> 

    ![](./images/running-the-app/new-tenant.png)

4. <span data-ttu-id="9b04d-126">Kattintson a **Create** (Létrehozás) gombra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-126">Click **Create**.</span></span> <span data-ttu-id="9b04d-127">Az új könyvtár létrehozása néhány percig is eltarthat.</span><span class="sxs-lookup"><span data-stu-id="9b04d-127">It may take a few minutes to create the new directory.</span></span>

<span data-ttu-id="9b04d-128">A végpont forgatókönyv végrehajtásához egy második Azure AD-címtár képviseli, amelyet az alkalmazás előfizet lesz szüksége.</span><span class="sxs-lookup"><span data-stu-id="9b04d-128">To complete the end-to-end scenario, you'll need a second Azure AD directory to represent a customer that signs up for the application.</span></span> <span data-ttu-id="9b04d-129">Az alapértelmezett Azure AD-címtár (nem Dejójáték) használja, vagy hozzon létre egy új könyvtárat erre a célra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-129">You can use your default Azure AD directory (not Tailspin), or create a new directory for this purpose.</span></span> <span data-ttu-id="9b04d-130">A példákban a Contoso a fiktív ügyfélként használjuk.</span><span class="sxs-lookup"><span data-stu-id="9b04d-130">In the examples, we use Contoso as the fictitious customer.</span></span>

## <a name="register-the-surveys-web-api"></a><span data-ttu-id="9b04d-131">Regisztrálja a felmérések webes API-k</span><span class="sxs-lookup"><span data-stu-id="9b04d-131">Register the Surveys web API</span></span> 

1. <span data-ttu-id="9b04d-132">Az a [Azure-portálon][portal], kiválasztja a fiókját a portál jobb felső sarkában az új Dejójáték könyvtár váltani.</span><span class="sxs-lookup"><span data-stu-id="9b04d-132">In the [Azure portal][portal], switch to the new Tailspin directory by selecting your account in the top right corner of the portal.</span></span>

2. <span data-ttu-id="9b04d-133">A bal oldali navigációs ablaktábláján válassza **Azure Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-133">In the left-hand navigation pane, choose **Azure Active Directory**.</span></span> 

3. <span data-ttu-id="9b04d-134">Kattintson a **App regisztrációk** > **új alkalmazás regisztrációja**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-134">Click **App registrations** > **New application registration**.</span></span>

4. <span data-ttu-id="9b04d-135">Az a **létrehozása** panelen adja meg a következő adatokat:</span><span class="sxs-lookup"><span data-stu-id="9b04d-135">In the **Create** blade, enter the following information:</span></span>

   - <span data-ttu-id="9b04d-136">**Név**: `Surveys.WebAPI`</span><span class="sxs-lookup"><span data-stu-id="9b04d-136">**Name**: `Surveys.WebAPI`</span></span>

   - <span data-ttu-id="9b04d-137">**Az alkalmazástípus**: `Web app / API`</span><span class="sxs-lookup"><span data-stu-id="9b04d-137">**Application type**: `Web app / API`</span></span>

   - <span data-ttu-id="9b04d-138">**Bejelentkezés URL-cím**: `https://localhost:44301/`</span><span class="sxs-lookup"><span data-stu-id="9b04d-138">**Sign-on URL**: `https://localhost:44301/`</span></span>
   
   ![](./images/running-the-app/register-web-api.png) 

5. <span data-ttu-id="9b04d-139">Kattintson a **Create** (Létrehozás) gombra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-139">Click **Create**.</span></span>

6. <span data-ttu-id="9b04d-140">Az a **App regisztrációk** panelen, jelölje be az új **Surveys.WebAPI** alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="9b04d-140">In the **App registrations** blade, select the new **Surveys.WebAPI** application.</span></span>
 
7. <span data-ttu-id="9b04d-141">Kattintson a **Tulajdonságok** elemre.</span><span class="sxs-lookup"><span data-stu-id="9b04d-141">Click **Properties**.</span></span>

8. <span data-ttu-id="9b04d-142">Az a **App ID URI** szerkesztési mezőhöz, adja meg `https://<domain>/surveys.webapi`, ahol `<domain>` annak a könyvtárnak a tartománynév.</span><span class="sxs-lookup"><span data-stu-id="9b04d-142">In the **App ID URI** edit box, enter `https://<domain>/surveys.webapi`, where `<domain>` is the domain name of the directory.</span></span> <span data-ttu-id="9b04d-143">Például:`https://tailspin.onmicrosoft.com/surveys.webapi`</span><span class="sxs-lookup"><span data-stu-id="9b04d-143">For example: `https://tailspin.onmicrosoft.com/surveys.webapi`</span></span>

    ![Beállítások](./images/running-the-app/settings.png)

9. <span data-ttu-id="9b04d-145">Állítsa be **Multi-központjaként** való **Igen**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-145">Set **Multi-tenanted** to **YES**.</span></span>

10. <span data-ttu-id="9b04d-146">Kattintson a **Save** (Mentés) gombra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-146">Click **Save**.</span></span>

## <a name="register-the-surveys-web-app"></a><span data-ttu-id="9b04d-147">A felmérések webes alkalmazás regisztrálása</span><span class="sxs-lookup"><span data-stu-id="9b04d-147">Register the Surveys web app</span></span> 

1. <span data-ttu-id="9b04d-148">Lépjen vissza a **App regisztrációk** panelen, majd kattintson **új alkalmazás regisztrációja**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-148">Navigate back to the **App registrations** blade, and click **New application registration**.</span></span>

2. <span data-ttu-id="9b04d-149">Az a **létrehozása** panelen adja meg a következő adatokat:</span><span class="sxs-lookup"><span data-stu-id="9b04d-149">In the **Create** blade, enter the following information:</span></span>

   - <span data-ttu-id="9b04d-150">**Név**: `Surveys`</span><span class="sxs-lookup"><span data-stu-id="9b04d-150">**Name**: `Surveys`</span></span>
   - <span data-ttu-id="9b04d-151">**Az alkalmazástípus**: `Web app / API`</span><span class="sxs-lookup"><span data-stu-id="9b04d-151">**Application type**: `Web app / API`</span></span>
   - <span data-ttu-id="9b04d-152">**Bejelentkezés URL-cím**: `https://localhost:44300/`</span><span class="sxs-lookup"><span data-stu-id="9b04d-152">**Sign-on URL**: `https://localhost:44300/`</span></span>
   
   <span data-ttu-id="9b04d-153">Figyelje meg, hogy a bejelentkezési URL-cím rendelkezik-e a port számát a `Surveys.WebAPI` alkalmazást az előző lépésben.</span><span class="sxs-lookup"><span data-stu-id="9b04d-153">Notice that the sign-on URL has a different port number from the `Surveys.WebAPI` app in the previous step.</span></span>

3. <span data-ttu-id="9b04d-154">Kattintson a **Create** (Létrehozás) gombra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-154">Click **Create**.</span></span>
 
4. <span data-ttu-id="9b04d-155">Az a **App regisztrációk** panelen, jelölje be az új **felmérések** alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="9b04d-155">In the **App registrations** blade, select the new **Surveys** application.</span></span>
 
5. <span data-ttu-id="9b04d-156">Másolja át az azonosítót.</span><span class="sxs-lookup"><span data-stu-id="9b04d-156">Copy the application ID.</span></span> <span data-ttu-id="9b04d-157">Ezt később lesz szüksége.</span><span class="sxs-lookup"><span data-stu-id="9b04d-157">You will need this later.</span></span>

    ![](./images/running-the-app/application-id.png)

6. <span data-ttu-id="9b04d-158">Kattintson a **Tulajdonságok** elemre.</span><span class="sxs-lookup"><span data-stu-id="9b04d-158">Click **Properties**.</span></span>

7. <span data-ttu-id="9b04d-159">Az a **App ID URI** szerkesztési mezőhöz, adja meg `https://<domain>/surveys`, ahol `<domain>` annak a könyvtárnak a tartománynév.</span><span class="sxs-lookup"><span data-stu-id="9b04d-159">In the **App ID URI** edit box, enter `https://<domain>/surveys`, where `<domain>` is the domain name of the directory.</span></span> 

    ![Beállítások](./images/running-the-app/settings.png)

8. <span data-ttu-id="9b04d-161">Állítsa be **Multi-központjaként** való **Igen**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-161">Set **Multi-tenanted** to **YES**.</span></span>

9. <span data-ttu-id="9b04d-162">Kattintson a **Save** (Mentés) gombra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-162">Click **Save**.</span></span>

10. <span data-ttu-id="9b04d-163">Az a **beállítások** panelen kattintson a **válasz URL-címek**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-163">In the **Settings** blade, click **Reply URLs**.</span></span>
 
11. <span data-ttu-id="9b04d-164">Adja hozzá a következő válasz URL-CÍMEN: `https://localhost:44300/signin-oidc`.</span><span class="sxs-lookup"><span data-stu-id="9b04d-164">Add the following reply URL: `https://localhost:44300/signin-oidc`.</span></span>

12. <span data-ttu-id="9b04d-165">Kattintson a **Save** (Mentés) gombra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-165">Click **Save**.</span></span>

13. <span data-ttu-id="9b04d-166">A **API-hozzáférés**, kattintson a **kulcsok**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-166">Under **API ACCESS**, click **Keys**.</span></span>

14. <span data-ttu-id="9b04d-167">Adja meg a leírását, például `client secret`.</span><span class="sxs-lookup"><span data-stu-id="9b04d-167">Enter a description, such as `client secret`.</span></span>

15. <span data-ttu-id="9b04d-168">Az a **válasszon időtartama** legördülő menüben válasszon ki **1 év**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-168">In the **Select Duration** dropdown, select **1 year**.</span></span> 

16. <span data-ttu-id="9b04d-169">Kattintson a **Save** (Mentés) gombra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-169">Click **Save**.</span></span> <span data-ttu-id="9b04d-170">A kulcs mentésekor jön létre.</span><span class="sxs-lookup"><span data-stu-id="9b04d-170">The key will be generated when you save.</span></span>

17. <span data-ttu-id="9b04d-171">Mielőtt ezt a panelt elhagyni, másolja a kulcsnak az értéke.</span><span class="sxs-lookup"><span data-stu-id="9b04d-171">Before you navigate away from this blade, copy the value of the key.</span></span>

    > [!NOTE] 
    > <span data-ttu-id="9b04d-172">A kulcs nem fog megjelenni újra után elnavigál a panelt.</span><span class="sxs-lookup"><span data-stu-id="9b04d-172">The key won't be visible again after you navigate away from the blade.</span></span> 

18. <span data-ttu-id="9b04d-173">A **API-hozzáférés**, kattintson a **szükséges engedélyek**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-173">Under **API ACCESS**, click **Required permissions**.</span></span>

19. <span data-ttu-id="9b04d-174">Kattintson a **hozzáadása** > **API kiválasztása**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-174">Click **Add** > **Select an API**.</span></span>

20. <span data-ttu-id="9b04d-175">A keresőmezőbe, keresse meg a `Surveys.WebAPI`.</span><span class="sxs-lookup"><span data-stu-id="9b04d-175">In the search box, search for `Surveys.WebAPI`.</span></span>

    ![Engedélyek](./images/running-the-app/permissions.png)

21. <span data-ttu-id="9b04d-177">Válassza ki `Surveys.WebAPI` kattintson **válasszon**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-177">Select `Surveys.WebAPI` and click **Select**.</span></span>

22. <span data-ttu-id="9b04d-178">A **delegált engedélyek**, ellenőrizze **hozzáférés Surveys.WebAPI**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-178">Under **Delegated Permissions**, check **Access Surveys.WebAPI**.</span></span>

    ![Engedélyek delegálás beállítása](./images/running-the-app/delegated-permissions.png)

23. <span data-ttu-id="9b04d-180">Kattintson a **válasszon** > **végzett**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-180">Click **Select** > **Done**.</span></span>


## <a name="update-the-application-manifests"></a><span data-ttu-id="9b04d-181">Az alkalmazásjegyzékeknek frissítése</span><span class="sxs-lookup"><span data-stu-id="9b04d-181">Update the application manifests</span></span>

1. <span data-ttu-id="9b04d-182">Lépjen vissza a **beállítások** panelen a `Surveys.WebAPI` alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="9b04d-182">Navigate back to the **Settings** blade for the `Surveys.WebAPI` app.</span></span>

2. <span data-ttu-id="9b04d-183">Kattintson a **Manifest** > **szerkesztése**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-183">Click **Manifest** > **Edit**.</span></span>

    ![](./images/running-the-app/manifest.png)
 
3. <span data-ttu-id="9b04d-184">Adja hozzá a következő JSON a `appRoles` elemet.</span><span class="sxs-lookup"><span data-stu-id="9b04d-184">Add the following JSON to the `appRoles` element.</span></span> <span data-ttu-id="9b04d-185">Új GUID azonosítóját készítése a `id` tulajdonságok.</span><span class="sxs-lookup"><span data-stu-id="9b04d-185">Generate new GUIDs for the `id` properties.</span></span>

   ```json
   {
     "allowedMemberTypes": ["User"],
     "description": "Creators can create surveys",
     "displayName": "SurveyCreator",
     "id": "<Generate a new GUID. Example: 1b4f816e-5eaf-48b9-8613-7923830595ad>",
     "isEnabled": true,
     "value": "SurveyCreator"
   },
   {
     "allowedMemberTypes": ["User"],
     "description": "Administrators can manage the surveys in their tenant",
     "displayName": "SurveyAdmin",
     "id": "<Generate a new GUID>",  
     "isEnabled": true,
     "value": "SurveyAdmin"
   }
   ```

4. <span data-ttu-id="9b04d-186">Az a `knownClientApplications` tulajdonság, vegye fel a felmérések webes alkalmazás, amely korábban a felmérések alkalmazás regisztrálásakor kapott Alkalmazásazonosító.</span><span class="sxs-lookup"><span data-stu-id="9b04d-186">In the `knownClientApplications` property, add the application ID for the Surveys web application, which you got when you registered the Surveys application earlier.</span></span> <span data-ttu-id="9b04d-187">Példa:</span><span class="sxs-lookup"><span data-stu-id="9b04d-187">For example:</span></span>

   ```json
   "knownClientApplications": ["be2cea23-aa0e-4e98-8b21-2963d494912e"],
   ```

   <span data-ttu-id="9b04d-188">Ezt a beállítást a felmérések alkalmazás hozzáadása a webes API hívásához jogosult ügyfelek listáját.</span><span class="sxs-lookup"><span data-stu-id="9b04d-188">This setting adds the Surveys app to the list of clients authorized to call the web API.</span></span>

5. <span data-ttu-id="9b04d-189">Kattintson a **Save** (Mentés) gombra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-189">Click **Save**.</span></span>

<span data-ttu-id="9b04d-190">Most ismételje meg a felmérések alkalmazást ugyanezekkel a lépésekkel kivéve ne adjon hozzá egy bejegyzést a `knownClientApplications`.</span><span class="sxs-lookup"><span data-stu-id="9b04d-190">Now repeat the same steps for the Surveys app, except do not add an entry for `knownClientApplications`.</span></span> <span data-ttu-id="9b04d-191">Használja az ugyanazon szerepkör-definíciók, de új GUID azonosítók előállításához.</span><span class="sxs-lookup"><span data-stu-id="9b04d-191">Use the same role definitions, but generate new GUIDs for the IDs.</span></span>

## <a name="create-a-new-redis-cache-instance"></a><span data-ttu-id="9b04d-192">Hozzon létre egy új Redis Cache példányt</span><span class="sxs-lookup"><span data-stu-id="9b04d-192">Create a new Redis Cache instance</span></span>

<span data-ttu-id="9b04d-193">A felmérések alkalmazás Redis gyorsítótár OAuth 2 jogkivonatot használja.</span><span class="sxs-lookup"><span data-stu-id="9b04d-193">The Surveys application uses Redis to cache OAuth 2 access tokens.</span></span> <span data-ttu-id="9b04d-194">A gyorsítótár létrehozásához:</span><span class="sxs-lookup"><span data-stu-id="9b04d-194">To create the cache:</span></span>

1.  <span data-ttu-id="9b04d-195">Ugrás a [Azure Portal](https://portal.azure.com) kattintson **új** > **adatbázisok** > **Redis Cache**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-195">Go to [Azure Portal](https://portal.azure.com) and click **New** > **Databases** > **Redis Cache**.</span></span>

2.  <span data-ttu-id="9b04d-196">Töltse ki a szükséges információt, beleértve a DNS-név, az erőforráscsoport, a hely, IP-címek.</span><span class="sxs-lookup"><span data-stu-id="9b04d-196">Fill in the required information, including DNS name, resource group, location, and pricing tier.</span></span> <span data-ttu-id="9b04d-197">Hozzon létre egy új erőforráscsoportot, vagy használjon egy meglévő erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="9b04d-197">You can create a new resource group or use an existing resource group.</span></span>

3. <span data-ttu-id="9b04d-198">Kattintson a **Create** (Létrehozás) gombra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-198">Click **Create**.</span></span>

4. <span data-ttu-id="9b04d-199">A Resis gyorsítótár létrehozása után nyissa meg az erőforrás a portálon.</span><span class="sxs-lookup"><span data-stu-id="9b04d-199">After the Resis cache is created, navigate to the resource in the portal.</span></span>

5. <span data-ttu-id="9b04d-200">Kattintson a **hívóbetűk** és másolja az elsődleges kulcsot.</span><span class="sxs-lookup"><span data-stu-id="9b04d-200">Click **Access keys** and copy the primary key.</span></span>

<span data-ttu-id="9b04d-201">A Redis gyorsítótár létrehozásával kapcsolatos további információkért lásd: [használata Azure Redis Cache hogyan](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache).</span><span class="sxs-lookup"><span data-stu-id="9b04d-201">For more information about creating a Redis cache, see [How to Use Azure Redis Cache](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache).</span></span>

## <a name="set-application-secrets"></a><span data-ttu-id="9b04d-202">Set alkalmazás titkos kulcsok</span><span class="sxs-lookup"><span data-stu-id="9b04d-202">Set application secrets</span></span>

1.  <span data-ttu-id="9b04d-203">Nyissa meg a Tailspin.Surveys megoldást a Visual Studióban.</span><span class="sxs-lookup"><span data-stu-id="9b04d-203">Open the Tailspin.Surveys solution in Visual Studio.</span></span>

2.  <span data-ttu-id="9b04d-204">A Megoldáskezelőben kattintson a jobb gombbal a Tailspin.Surveys.Web projektet, és válassza ki **felhasználói kulcsok kezelése**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-204">In Solution Explorer, right-click the Tailspin.Surveys.Web project and select **Manage User Secrets**.</span></span>

3.  <span data-ttu-id="9b04d-205">A secrets.json fájl illessze be az alábbiakat:</span><span class="sxs-lookup"><span data-stu-id="9b04d-205">In the secrets.json file, paste in the following:</span></span>
    
    ```json
    {
      "AzureAd": {
        "ClientId": "<Surveys application ID>",
        "ClientSecret": "<Surveys app client secret>",
        "PostLogoutRedirectUri": "https://localhost:44300/",
        "WebApiResourceId": "<Surveys.WebAPI app ID URI>"
      },
      "Redis": {
        "Configuration": "<Redis DNS name>.redis.cache.windows.net,password=<Redis primary key>,ssl=true"
      }
    }
    ```
   
    <span data-ttu-id="9b04d-206">Cserélje le a következőképpen csúcsos zárójelek megjelenő elemek:</span><span class="sxs-lookup"><span data-stu-id="9b04d-206">Replace the items shown in angle brackets, as follows:</span></span>

    - <span data-ttu-id="9b04d-207">`AzureAd:ClientId`: A felmérések alkalmazás az alkalmazás azonosítója.</span><span class="sxs-lookup"><span data-stu-id="9b04d-207">`AzureAd:ClientId`: The application ID of the Surveys app.</span></span>
    - <span data-ttu-id="9b04d-208">`AzureAd:ClientSecret`: A kulcs az Azure ad-ben a felmérések alkalmazás regisztrálásakor létrehozott.</span><span class="sxs-lookup"><span data-stu-id="9b04d-208">`AzureAd:ClientSecret`: The key that you generated when you registered the Surveys application in Azure AD.</span></span>
    - <span data-ttu-id="9b04d-209">`AzureAd:WebApiResourceId`: A App ID URI Azure AD-ben a Surveys.WebAPI alkalmazás létrehozásakor megadott.</span><span class="sxs-lookup"><span data-stu-id="9b04d-209">`AzureAd:WebApiResourceId`: The App ID URI that you specified when you created the Surveys.WebAPI application in Azure AD.</span></span> <span data-ttu-id="9b04d-210">Az űrlap rendelkeznie kell `https://<directory>.onmicrosoft.com/surveys.webapi`</span><span class="sxs-lookup"><span data-stu-id="9b04d-210">It should have the form `https://<directory>.onmicrosoft.com/surveys.webapi`</span></span>
    - <span data-ttu-id="9b04d-211">`Redis:Configuration`: Hozhat létre. Ez a karakterlánc a DNS neve alapján a Redis gyorsítótárt és az elsődleges elérési kulcsot.</span><span class="sxs-lookup"><span data-stu-id="9b04d-211">`Redis:Configuration`: Build this string from the DNS name of the Redis cache and the primary access key.</span></span> <span data-ttu-id="9b04d-212">For example, "tailspin.redis.cache.windows.net,password=2h5tBxxx,ssl=true".</span><span class="sxs-lookup"><span data-stu-id="9b04d-212">For example, "tailspin.redis.cache.windows.net,password=2h5tBxxx,ssl=true".</span></span>

4.  <span data-ttu-id="9b04d-213">Mentse a frissített secrets.json fájlt.</span><span class="sxs-lookup"><span data-stu-id="9b04d-213">Save the updated secrets.json file.</span></span>

5.  <span data-ttu-id="9b04d-214">Ismételje meg ezeket a lépéseket a Tailspin.Surveys.WebAPI projekt esetében, de illessze be a következő secrets.json.</span><span class="sxs-lookup"><span data-stu-id="9b04d-214">Repeat these steps for the Tailspin.Surveys.WebAPI project, but paste the following into secrets.json.</span></span> <span data-ttu-id="9b04d-215">Cserélje le azt korábban a csúcsos zárójelek elemeket.</span><span class="sxs-lookup"><span data-stu-id="9b04d-215">Replace the items in angle brackets, as before.</span></span>

    ```json
    {
      "AzureAd": {
        "WebApiResourceId": "<Surveys.WebAPI app ID URI>"
      },
      "Redis": {
        "Configuration": "<Redis DNS name>.redis.cache.windows.net,password=<Redis primary key>,ssl=true"
      }
    }
    ```

## <a name="initialize-the-database"></a><span data-ttu-id="9b04d-216">Az adatbázis inicializálása</span><span class="sxs-lookup"><span data-stu-id="9b04d-216">Initialize the database</span></span>

<span data-ttu-id="9b04d-217">Ebben a lépésben szüksége lesz az Entity Framework 7 LocalDB használata egy helyi SQL-adatbázis létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="9b04d-217">In this step, you will use Entity Framework 7 to create a local SQL database, using LocalDB.</span></span>

1.  <span data-ttu-id="9b04d-218">Nyisson meg egy parancsablakot</span><span class="sxs-lookup"><span data-stu-id="9b04d-218">Open a command window</span></span>

2.  <span data-ttu-id="9b04d-219">Nyissa meg a Tailspin.Surveys.Data projekt.</span><span class="sxs-lookup"><span data-stu-id="9b04d-219">Navigate to the Tailspin.Surveys.Data project.</span></span>

3.  <span data-ttu-id="9b04d-220">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="9b04d-220">Run the following command:</span></span>

    ```
    dotnet ef database update --startup-project ..\Tailspin.Surveys.Web
    ```
    
## <a name="run-the-application"></a><span data-ttu-id="9b04d-221">Az alkalmazás futtatása</span><span class="sxs-lookup"><span data-stu-id="9b04d-221">Run the application</span></span>

<span data-ttu-id="9b04d-222">Futtassa az alkalmazást, indítsa el a Tailspin.Surveys.Web és a Tailspin.Surveys.WebAPI projektek.</span><span class="sxs-lookup"><span data-stu-id="9b04d-222">To run the application, start both the Tailspin.Surveys.Web and Tailspin.Surveys.WebAPI projects.</span></span>

<span data-ttu-id="9b04d-223">A Visual Studio segítségével mindkét projekt futtatásához automatikusan F5, a következőképpen állíthatja be:</span><span class="sxs-lookup"><span data-stu-id="9b04d-223">You can set Visual Studio to run both projects automatically on F5, as follows:</span></span>

1.  <span data-ttu-id="9b04d-224">A Megoldáskezelőben kattintson a jobb gombbal a megoldás, és kattintson a **indítási projektek beállítása**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-224">In Solution Explorer, right-click the solution and click **Set Startup Projects**.</span></span>
2.  <span data-ttu-id="9b04d-225">Válassza ki **több kezdőprojekt**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-225">Select **Multiple startup projects**.</span></span>
3.  <span data-ttu-id="9b04d-226">Állítsa be **művelet** = **Start** Tailspin.Surveys.Web és Tailspin.Surveys.WebAPI projektek.</span><span class="sxs-lookup"><span data-stu-id="9b04d-226">Set **Action** = **Start** for the Tailspin.Surveys.Web and Tailspin.Surveys.WebAPI projects.</span></span>

## <a name="sign-up-a-new-tenant"></a><span data-ttu-id="9b04d-227">Iratkozzon fel egy új bérlőt</span><span class="sxs-lookup"><span data-stu-id="9b04d-227">Sign up a new tenant</span></span>

<span data-ttu-id="9b04d-228">Az alkalmazás indításakor nem jelentkezett be, így a kezdőlapon:</span><span class="sxs-lookup"><span data-stu-id="9b04d-228">When the application starts, you are not signed in, so you see the welcome page:</span></span>

![Kezdőlap](./images/running-the-app/screenshot1.png)

<span data-ttu-id="9b04d-230">A szervezet feliratkozáshoz:</span><span class="sxs-lookup"><span data-stu-id="9b04d-230">To sign up an organization:</span></span>

1. <span data-ttu-id="9b04d-231">Kattintson a **regisztrálja a vállalat Dejójáték**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-231">Click **Enroll your company in Tailspin**.</span></span>
2. <span data-ttu-id="9b04d-232">Jelentkezzen be az Azure AD-címtárral, amely a szervezet a felmérések alkalmazással jelöli.</span><span class="sxs-lookup"><span data-stu-id="9b04d-232">Sign in to the Azure AD directory that represents the organization using the Surveys app.</span></span> <span data-ttu-id="9b04d-233">Be kell jelentkeznie rendszergazda felhasználóként.</span><span class="sxs-lookup"><span data-stu-id="9b04d-233">You must sign in as an admin user.</span></span>
3. <span data-ttu-id="9b04d-234">Fogadja el a jóváhagyási kérése.</span><span class="sxs-lookup"><span data-stu-id="9b04d-234">Accept the consent prompt.</span></span>

<span data-ttu-id="9b04d-235">Az alkalmazás a bérlő regisztrálja, és majd kijelentkezik meg. Az alkalmazás kijelentkezik, mert később szüksége lesz a szerepkörökhöz beállítása az Azure AD, az alkalmazás használata előtt.</span><span class="sxs-lookup"><span data-stu-id="9b04d-235">The application registers the tenant, and then signs you out. The app signs you out because you need to set up the application roles in Azure AD, before using the application.</span></span>

![Regisztráció után](./images/running-the-app/screenshot2.png)

## <a name="assign-application-roles"></a><span data-ttu-id="9b04d-237">Alkalmazás-szerepkörök hozzárendelése</span><span class="sxs-lookup"><span data-stu-id="9b04d-237">Assign application roles</span></span>

<span data-ttu-id="9b04d-238">Amikor a bérlő előfizet, AD a bérlői rendszergazda felhasználók szerepkörökhöz kell rendelni.</span><span class="sxs-lookup"><span data-stu-id="9b04d-238">When a tenant signs up, an AD admin for the tenant must assign application roles to users.</span></span>


1. <span data-ttu-id="9b04d-239">Az a [Azure-portálon][portal], az Azure AD-címtár, amellyel iratkozzon fel a felmérések app váltani.</span><span class="sxs-lookup"><span data-stu-id="9b04d-239">In the [Azure portal][portal], switch to the Azure AD directory that you used to sign up for the Surveys app.</span></span> 

2. <span data-ttu-id="9b04d-240">A bal oldali navigációs ablaktábláján válassza **Azure Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-240">In the left-hand navigation pane, choose **Azure Active Directory**.</span></span> 

3. <span data-ttu-id="9b04d-241">Kattintson a **vállalati alkalmazások** > **összes alkalmazás**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-241">Click **Enterprise applications** > **All applications**.</span></span> <span data-ttu-id="9b04d-242">A portál felsorolja `Survey` és `Survey.WebAPI`.</span><span class="sxs-lookup"><span data-stu-id="9b04d-242">The portal will list `Survey` and `Survey.WebAPI`.</span></span> <span data-ttu-id="9b04d-243">Ha nem, akkor győződjön meg arról, hogy a regisztrációs folyamat befejeződött-e.</span><span class="sxs-lookup"><span data-stu-id="9b04d-243">If not, make sure that you completed the sign up process.</span></span>

4.  <span data-ttu-id="9b04d-244">Kattintson az felmérések alkalmazásra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-244">Click on the Surveys application.</span></span>

5.  <span data-ttu-id="9b04d-245">Kattintson a **felhasználók és csoportok**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-245">Click **Users and Groups**.</span></span>

4.  <span data-ttu-id="9b04d-246">Kattintson a **felhasználó hozzáadása**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-246">Click **Add user**.</span></span>

5.  <span data-ttu-id="9b04d-247">Ha a prémium szintű Azure AD, kattintson **felhasználók és csoportok**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-247">If you have Azure AD Premium, click **Users and groups**.</span></span> <span data-ttu-id="9b04d-248">Ellenkező esetben kattintson a **felhasználók**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-248">Otherwise, click **Users**.</span></span> <span data-ttu-id="9b04d-249">(Az Azure AD Premium szerepkört rendel egy csoport szükséges.)</span><span class="sxs-lookup"><span data-stu-id="9b04d-249">(Assigning a role to a group requires Azure AD Premium.)</span></span>

6. <span data-ttu-id="9b04d-250">Jelöljön ki egy vagy több felhasználót, majd **válasszon**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-250">Select one or more users and click **Select**.</span></span>

    ![Felhasználó vagy csoport kiválasztása](./images/running-the-app/select-user-or-group.png)

6.  <span data-ttu-id="9b04d-252">Válassza ki a szerepkört, és kattintson a **válasszon**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-252">Select the role and click **Select**.</span></span>

    ![Felhasználó vagy csoport kiválasztása](./images/running-the-app/select-role.png)

7.  <span data-ttu-id="9b04d-254">Kattintson a **Hozzárendelés** gombra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-254">Click **Assign**.</span></span>

<span data-ttu-id="9b04d-255">Ismételje meg a azonos Survey.WebAPI alkalmazásához szerepköröket hozzárendelni.</span><span class="sxs-lookup"><span data-stu-id="9b04d-255">Repeat the same steps to assign roles for the Survey.WebAPI application.</span></span>

> <span data-ttu-id="9b04d-256">Fontos: A felhasználó mindig meg kell adni a felmérést és a Survey.WebAPI azonos szerepkörök.</span><span class="sxs-lookup"><span data-stu-id="9b04d-256">Important: A user should always have the same roles in both Survey and Survey.WebAPI.</span></span> <span data-ttu-id="9b04d-257">Ellenkező esetben a felhasználónak kell inkonzisztens engedélyek, a webes API 403-as (tiltott) hibákhoz vezethetnek.</span><span class="sxs-lookup"><span data-stu-id="9b04d-257">Otherwise, the user will have inconsistent permissions, which may lead to 403 (Forbidden) errors from the Web API.</span></span>

<span data-ttu-id="9b04d-258">Most lépjen vissza az alkalmazást, és jelentkezzen be újra.</span><span class="sxs-lookup"><span data-stu-id="9b04d-258">Now go back to the app and sign in again.</span></span> <span data-ttu-id="9b04d-259">Kattintson a **a felmérések**.</span><span class="sxs-lookup"><span data-stu-id="9b04d-259">Click **My Surveys**.</span></span> <span data-ttu-id="9b04d-260">Ha a felhasználó a SurveyAdmin vagy SurveyCreator szerepkör van hozzárendelve, megjelenik egy **felmérés létrehozása** gomb, amely azt jelzi, hogy a felhasználó rendelkezik-e egy új felmérés létrehozásához szükséges engedélyek.</span><span class="sxs-lookup"><span data-stu-id="9b04d-260">If the user is assigned to the SurveyAdmin or SurveyCreator role, you will see a **Create Survey** button, indicating that the user has permissions to create a new survey.</span></span>

![A felmérések](./images/running-the-app/screenshot3.png)


<!-- links -->

[portal]: https://portal.azure.com
[VS2017]: https://www.visualstudio.com/vs/
