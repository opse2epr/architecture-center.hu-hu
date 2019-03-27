---
title: A Surveys alkalmazás futtatása
description: Hogyan lehet a Surveys mintaalkalmazás helyi futtatásához.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: bdc2f4da6dd151cf395bee700e6a44e827c1e4f9
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299245"
---
# <a name="run-the-surveys-application"></a><span data-ttu-id="15d9a-103">A Surveys alkalmazás futtatása</span><span class="sxs-lookup"><span data-stu-id="15d9a-103">Run the Surveys application</span></span>

<span data-ttu-id="15d9a-104">Ez a cikk azt ismerteti, hogyan futtathat a [Tailspin Surveys](./tailspin.md) alkalmazást helyileg, a Visual Studióból.</span><span class="sxs-lookup"><span data-stu-id="15d9a-104">This article describes how to run the [Tailspin Surveys](./tailspin.md) application locally, from Visual Studio.</span></span> <span data-ttu-id="15d9a-105">Ezeket a lépéseket az Azure-ban az alkalmazás nem telepít.</span><span class="sxs-lookup"><span data-stu-id="15d9a-105">In these steps, you won't deploy the application to Azure.</span></span> <span data-ttu-id="15d9a-106">Azonban szüksége lesz egy Azure-erőforrások létrehozása &mdash; egy Azure Active Directory (Azure AD) címtár és a egy Redis gyorsítótárhoz.</span><span class="sxs-lookup"><span data-stu-id="15d9a-106">However, you will need to create some Azure resources &mdash; an Azure Active Directory (Azure AD) directory and a Redis cache.</span></span>

<span data-ttu-id="15d9a-107">A következő lépések összefoglalása:</span><span class="sxs-lookup"><span data-stu-id="15d9a-107">Here is a summary of the steps:</span></span>

1. <span data-ttu-id="15d9a-108">A Tailspin fiktív hozzon létre egy Azure AD-címtár (bérlő).</span><span class="sxs-lookup"><span data-stu-id="15d9a-108">Create an Azure AD directory (tenant) for the fictitious Tailspin company.</span></span>
2. <span data-ttu-id="15d9a-109">A Surveys alkalmazás és a háttéralkalmazás webes API regisztrálása az Azure ad-ben.</span><span class="sxs-lookup"><span data-stu-id="15d9a-109">Register the Surveys application and the backend web API with Azure AD.</span></span>
3. <span data-ttu-id="15d9a-110">Hozzon létre egy Azure Redis Cache-példányhoz.</span><span class="sxs-lookup"><span data-stu-id="15d9a-110">Create an Azure Redis Cache instance.</span></span>
4. <span data-ttu-id="15d9a-111">Az Alkalmazásbeállítások konfigurálása, és hozzon létre egy helyi adatbázist.</span><span class="sxs-lookup"><span data-stu-id="15d9a-111">Configure application settings and create a local database.</span></span>
5. <span data-ttu-id="15d9a-112">Futtassa az alkalmazást, és regisztráljon egy új bérlőt.</span><span class="sxs-lookup"><span data-stu-id="15d9a-112">Run the application and sign up a new tenant.</span></span>
6. <span data-ttu-id="15d9a-113">Alkalmazás-szerepkörök hozzáadása a felhasználók számára.</span><span class="sxs-lookup"><span data-stu-id="15d9a-113">Add application roles to users.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="15d9a-114">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="15d9a-114">Prerequisites</span></span>

- <span data-ttu-id="15d9a-115">[A Visual Studio 2017] [ VS2017] együtt a [ASP.NET- és fejlesztési számítási feladatot](https://visualstudio.microsoft.com/vs/support/selecting-workloads-visual-studio-2017) telepítve</span><span class="sxs-lookup"><span data-stu-id="15d9a-115">[Visual Studio 2017][VS2017] with the [ASP.NET and web development workload](https://visualstudio.microsoft.com/vs/support/selecting-workloads-visual-studio-2017) installed</span></span>
- <span data-ttu-id="15d9a-116">[A Microsoft Azure](https://azure.microsoft.com) fiók</span><span class="sxs-lookup"><span data-stu-id="15d9a-116">[Microsoft Azure](https://azure.microsoft.com) account</span></span>

## <a name="create-the-tailspin-tenant"></a><span data-ttu-id="15d9a-117">A Tailspin bérlő létrehozása</span><span class="sxs-lookup"><span data-stu-id="15d9a-117">Create the Tailspin tenant</span></span>

<span data-ttu-id="15d9a-118">Tailspin a fiktív vállalat, amely futtatja a Surveys alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="15d9a-118">Tailspin is the fictitious company that hosts the Surveys application.</span></span> <span data-ttu-id="15d9a-119">Tailspin regisztrálni az alkalmazást a többi bérlő engedélyezése az Azure AD.</span><span class="sxs-lookup"><span data-stu-id="15d9a-119">Tailspin uses Azure AD to enable other tenants to register with the app.</span></span> <span data-ttu-id="15d9a-120">Ezek az ügyfelek ezután használhatja az Azure AD hitelesítő adataikkal bejelentkeznie az alkalmazásba.</span><span class="sxs-lookup"><span data-stu-id="15d9a-120">Those customers can then use their Azure AD credentials to sign into the app.</span></span>

<span data-ttu-id="15d9a-121">Ebben a lépésben az Azure AD-címtárat a Tailspin fog létrehozni.</span><span class="sxs-lookup"><span data-stu-id="15d9a-121">In this step, you'll create an Azure AD directory for Tailspin.</span></span>

1. <span data-ttu-id="15d9a-122">Jelentkezzen be az [Azure Portalra][portal].</span><span class="sxs-lookup"><span data-stu-id="15d9a-122">Sign into the [Azure portal][portal].</span></span>

2. <span data-ttu-id="15d9a-123">Kattintson a **+ erőforrás létrehozása** > **identitás** > **az Azure Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-123">Click **+ Create a Resource** > **Identity** > **Azure Active Directory**.</span></span>

3. <span data-ttu-id="15d9a-124">Adja meg `Tailspin` a szervezet neve, és adjon meg tartománynevet.</span><span class="sxs-lookup"><span data-stu-id="15d9a-124">Enter `Tailspin` for the organization name, and enter a domain name.</span></span> <span data-ttu-id="15d9a-125">A tartomány neve lesz a képernyő `xxxx.onmicrosoft.com` és globálisan egyedinek kell lennie.</span><span class="sxs-lookup"><span data-stu-id="15d9a-125">The domain name will have the form `xxxx.onmicrosoft.com` and must be globally unique.</span></span>

    ![Könyvtár párbeszédpanel létrehozása](./images/running-the-app/new-tenant.png)

4. <span data-ttu-id="15d9a-127">Kattintson a **Create** (Létrehozás) gombra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-127">Click **Create**.</span></span> <span data-ttu-id="15d9a-128">Az új könyvtár létrehozása néhány percet igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="15d9a-128">It may take a few minutes to create the new directory.</span></span>

<span data-ttu-id="15d9a-129">A végpontok közötti forgatókönyv végrehajtásához szüksége lesz egy második Azure AD-címtár, amely jelöli, amelyet az alkalmazás regisztrál.</span><span class="sxs-lookup"><span data-stu-id="15d9a-129">To complete the end-to-end scenario, you'll need a second Azure AD directory to represent a customer that signs up for the application.</span></span> <span data-ttu-id="15d9a-130">Használja az alapértelmezett Azure AD-címtár (nem a Tailspin), vagy hozzon létre egy új könyvtárat erre a célra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-130">You can use your default Azure AD directory (not Tailspin), or create a new directory for this purpose.</span></span> <span data-ttu-id="15d9a-131">A példákban a Contoso a fiktív ügyfélként használjuk.</span><span class="sxs-lookup"><span data-stu-id="15d9a-131">In the examples, we use Contoso as the fictitious customer.</span></span>

## <a name="register-the-surveys-web-api"></a><span data-ttu-id="15d9a-132">A felmérések webes API regisztrálása</span><span class="sxs-lookup"><span data-stu-id="15d9a-132">Register the Surveys web API</span></span>

1. <span data-ttu-id="15d9a-133">Az a [az Azure portal][portal], váltson az új Tailspin könyvtárat a portál jobb felső sarkában a fiók kiválasztásával.</span><span class="sxs-lookup"><span data-stu-id="15d9a-133">In the [Azure portal][portal], switch to the new Tailspin directory by selecting your account in the top right corner of the portal.</span></span>

2. <span data-ttu-id="15d9a-134">A bal oldali navigációs ablaktáblán válassza ki a **Azure Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-134">In the left-hand navigation pane, choose **Azure Active Directory**.</span></span>

3. <span data-ttu-id="15d9a-135">Kattintson a **alkalmazásregisztrációk** > **új alkalmazásregisztráció**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-135">Click **App registrations** > **New application registration**.</span></span>

4. <span data-ttu-id="15d9a-136">Az a **létrehozás** panelen adja meg a következőket:</span><span class="sxs-lookup"><span data-stu-id="15d9a-136">In the **Create** blade, enter the following information:</span></span>

   - <span data-ttu-id="15d9a-137">**Név**: `Surveys.WebAPI`</span><span class="sxs-lookup"><span data-stu-id="15d9a-137">**Name**: `Surveys.WebAPI`</span></span>

   - <span data-ttu-id="15d9a-138">**Az alkalmazástípus**: `Web app / API`</span><span class="sxs-lookup"><span data-stu-id="15d9a-138">**Application type**: `Web app / API`</span></span>

   - <span data-ttu-id="15d9a-139">**Bejelentkezés URL-cím**: `https://localhost:44301/`</span><span class="sxs-lookup"><span data-stu-id="15d9a-139">**Sign-on URL**: `https://localhost:44301/`</span></span>

   ![Képernyőfelvétel a webes API regisztrálása](./images/running-the-app/register-web-api.png)

5. <span data-ttu-id="15d9a-141">Kattintson a **Create** (Létrehozás) gombra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-141">Click **Create**.</span></span>

6. <span data-ttu-id="15d9a-142">Az a **alkalmazásregisztrációk** panelen válassza ki az új **Surveys.WebAPI** alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="15d9a-142">In the **App registrations** blade, select the new **Surveys.WebAPI** application.</span></span>

7. <span data-ttu-id="15d9a-143">Kattintson a **beállítások** > **tulajdonságok**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-143">Click **Settings** > **Properties**.</span></span>

8. <span data-ttu-id="15d9a-144">Az a **Alkalmazásazonosító URI-t** beviteli mező, adja meg `https://<domain>/surveys.webapi`, ahol `<domain>` tartományneve annak a címtárnak.</span><span class="sxs-lookup"><span data-stu-id="15d9a-144">In the **App ID URI** edit box, enter `https://<domain>/surveys.webapi`, where `<domain>` is the domain name of the directory.</span></span> <span data-ttu-id="15d9a-145">Például:`https://tailspin.onmicrosoft.com/surveys.webapi`</span><span class="sxs-lookup"><span data-stu-id="15d9a-145">For example: `https://tailspin.onmicrosoft.com/surveys.webapi`</span></span>

    ![Beállítások](./images/running-the-app/settings.png)

9. <span data-ttu-id="15d9a-147">Állítsa be **több-bérlős** való **Igen**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-147">Set **Multi-tenanted** to **YES**.</span></span>

10. <span data-ttu-id="15d9a-148">Kattintson a **Save** (Mentés) gombra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-148">Click **Save**.</span></span>

## <a name="register-the-surveys-web-app"></a><span data-ttu-id="15d9a-149">A felmérések webes alkalmazás regisztrálása</span><span class="sxs-lookup"><span data-stu-id="15d9a-149">Register the Surveys web app</span></span>

1. <span data-ttu-id="15d9a-150">Lépjen vissza a **alkalmazásregisztrációk** panelen, és kattintson **új alkalmazásregisztráció**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-150">Navigate back to the **App registrations** blade, and click **New application registration**.</span></span>

2. <span data-ttu-id="15d9a-151">Az a **létrehozás** panelen adja meg a következőket:</span><span class="sxs-lookup"><span data-stu-id="15d9a-151">In the **Create** blade, enter the following information:</span></span>

    - <span data-ttu-id="15d9a-152">**Név**: `Surveys`</span><span class="sxs-lookup"><span data-stu-id="15d9a-152">**Name**: `Surveys`</span></span>
    - <span data-ttu-id="15d9a-153">**Az alkalmazástípus**: `Web app / API`</span><span class="sxs-lookup"><span data-stu-id="15d9a-153">**Application type**: `Web app / API`</span></span>
    - <span data-ttu-id="15d9a-154">**Bejelentkezés URL-cím**: `https://localhost:44300/`</span><span class="sxs-lookup"><span data-stu-id="15d9a-154">**Sign-on URL**: `https://localhost:44300/`</span></span>

    <span data-ttu-id="15d9a-155">Figyelje meg, hogy a bejelentkezési URL-rendelkezik-e a port számát a `Surveys.WebAPI` alkalmazás az előző lépésben.</span><span class="sxs-lookup"><span data-stu-id="15d9a-155">Notice that the sign-on URL has a different port number from the `Surveys.WebAPI` app in the previous step.</span></span>

3. <span data-ttu-id="15d9a-156">Kattintson a **Create** (Létrehozás) gombra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-156">Click **Create**.</span></span>

4. <span data-ttu-id="15d9a-157">Az a **alkalmazásregisztrációk** panelen válassza ki az új **felmérések** alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="15d9a-157">In the **App registrations** blade, select the new **Surveys** application.</span></span>

5. <span data-ttu-id="15d9a-158">Másolja az alkalmazás azonosítója.</span><span class="sxs-lookup"><span data-stu-id="15d9a-158">Copy the application ID.</span></span> <span data-ttu-id="15d9a-159">Ez később lesz szüksége.</span><span class="sxs-lookup"><span data-stu-id="15d9a-159">You will need this later.</span></span>

    ![Képernyőkép az Alkalmazásazonosító másolása](./images/running-the-app/application-id.png)

6. <span data-ttu-id="15d9a-161">Kattintson a **Tulajdonságok** elemre.</span><span class="sxs-lookup"><span data-stu-id="15d9a-161">Click **Properties**.</span></span>

7. <span data-ttu-id="15d9a-162">Az a **Alkalmazásazonosító URI-t** beviteli mező, adja meg `https://<domain>/surveys`, ahol `<domain>` tartományneve annak a címtárnak.</span><span class="sxs-lookup"><span data-stu-id="15d9a-162">In the **App ID URI** edit box, enter `https://<domain>/surveys`, where `<domain>` is the domain name of the directory.</span></span>

    ![Beállítások](./images/running-the-app/settings.png)

8. <span data-ttu-id="15d9a-164">Állítsa be **több-bérlős** való **Igen**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-164">Set **Multi-tenanted** to **YES**.</span></span>

9. <span data-ttu-id="15d9a-165">Kattintson a **Save** (Mentés) gombra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-165">Click **Save**.</span></span>

10. <span data-ttu-id="15d9a-166">Az a **beállítások** panelen kattintson a **válasz URL-címek**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-166">In the **Settings** blade, click **Reply URLs**.</span></span>

11. <span data-ttu-id="15d9a-167">Adja hozzá a következő válasz URL-cím: `https://localhost:44300/signin-oidc`.</span><span class="sxs-lookup"><span data-stu-id="15d9a-167">Add the following reply URL: `https://localhost:44300/signin-oidc`.</span></span>

12. <span data-ttu-id="15d9a-168">Kattintson a **Save** (Mentés) gombra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-168">Click **Save**.</span></span>

13. <span data-ttu-id="15d9a-169">A **API-hozzáférés**, kattintson a **kulcsok**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-169">Under **API ACCESS**, click **Keys**.</span></span>

14. <span data-ttu-id="15d9a-170">Adja meg a leírását, például `client secret`.</span><span class="sxs-lookup"><span data-stu-id="15d9a-170">Enter a description, such as `client secret`.</span></span>

15. <span data-ttu-id="15d9a-171">Az a **kiválasztása időtartama** legördülő menüben válasszon ki **1 év**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-171">In the **Select Duration** dropdown, select **1 year**.</span></span>

16. <span data-ttu-id="15d9a-172">Kattintson a **Save** (Mentés) gombra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-172">Click **Save**.</span></span> <span data-ttu-id="15d9a-173">A kulcs jön létre, amikor menti.</span><span class="sxs-lookup"><span data-stu-id="15d9a-173">The key will be generated when you save.</span></span>

17. <span data-ttu-id="15d9a-174">Mielőtt ezt a panelt elhagyni, másolja a kulcs értékét.</span><span class="sxs-lookup"><span data-stu-id="15d9a-174">Before you navigate away from this blade, copy the value of the key.</span></span>

    > [!NOTE]
    > <span data-ttu-id="15d9a-175">A kulcs nem lesz látható újra után meg elhagyni a panelt.</span><span class="sxs-lookup"><span data-stu-id="15d9a-175">The key won't be visible again after you navigate away from the blade.</span></span>

18. <span data-ttu-id="15d9a-176">A **API-hozzáférés**, kattintson a **szükséges engedélyek**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-176">Under **API ACCESS**, click **Required permissions**.</span></span>

19. <span data-ttu-id="15d9a-177">Kattintson a **hozzáadása** > **API kiválasztása**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-177">Click **Add** > **Select an API**.</span></span>

20. <span data-ttu-id="15d9a-178">A keresőmezőbe keresése `Surveys.WebAPI`.</span><span class="sxs-lookup"><span data-stu-id="15d9a-178">In the search box, search for `Surveys.WebAPI`.</span></span>

    ![Engedélyek](./images/running-the-app/permissions.png)

21. <span data-ttu-id="15d9a-180">Válassza ki `Surveys.WebAPI` kattintson **kiválasztása**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-180">Select `Surveys.WebAPI` and click **Select**.</span></span>

22. <span data-ttu-id="15d9a-181">A **delegált engedélyek**, ellenőrizze **hozzáférés Surveys.WebAPI**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-181">Under **Delegated Permissions**, check **Access Surveys.WebAPI**.</span></span>

    ![Engedélyek beállítása meghatalmazott](./images/running-the-app/delegated-permissions.png)

23. <span data-ttu-id="15d9a-183">Kattintson a **Kiválasztás** > **Kész** gombra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-183">Click **Select** > **Done**.</span></span>

## <a name="update-the-application-manifests"></a><span data-ttu-id="15d9a-184">Frissítse az alkalmazásjegyzékeket</span><span class="sxs-lookup"><span data-stu-id="15d9a-184">Update the application manifests</span></span>

1. <span data-ttu-id="15d9a-185">Lépjen vissza a **beállítások** paneljén a `Surveys.WebAPI` alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="15d9a-185">Navigate back to the **Settings** blade for the `Surveys.WebAPI` app.</span></span>

2. <span data-ttu-id="15d9a-186">Kattintson a **Manifest** > **szerkesztése**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-186">Click **Manifest** > **Edit**.</span></span>

    ![Képernyőkép az alkalmazásjegyzék szerkesztése](./images/running-the-app/manifest.png)

3. <span data-ttu-id="15d9a-188">Adja hozzá a következő JSON-a `appRoles` elemet.</span><span class="sxs-lookup"><span data-stu-id="15d9a-188">Add the following JSON to the `appRoles` element.</span></span> <span data-ttu-id="15d9a-189">Hozzon létre új GUID-azonosítói a `id` tulajdonságait.</span><span class="sxs-lookup"><span data-stu-id="15d9a-189">Generate new GUIDs for the `id` properties.</span></span>

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

4. <span data-ttu-id="15d9a-190">Az a `knownClientApplications` tulajdonság, adja hozzá a felmérések webes alkalmazás, amely korábban a Surveys alkalmazás regisztrálásakor kapott Alkalmazásazonosító.</span><span class="sxs-lookup"><span data-stu-id="15d9a-190">In the `knownClientApplications` property, add the application ID for the Surveys web application, which you got when you registered the Surveys application earlier.</span></span> <span data-ttu-id="15d9a-191">Példa:</span><span class="sxs-lookup"><span data-stu-id="15d9a-191">For example:</span></span>

   ```json
   "knownClientApplications": ["be2cea23-aa0e-4e98-8b21-2963d494912e"],
   ```

   <span data-ttu-id="15d9a-192">Ez a beállítás a Surveys alkalmazás hozzáadása a webes API hívása jogosult ügyfelek listáját.</span><span class="sxs-lookup"><span data-stu-id="15d9a-192">This setting adds the Surveys app to the list of clients authorized to call the web API.</span></span>

5. <span data-ttu-id="15d9a-193">Kattintson a **Save** (Mentés) gombra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-193">Click **Save**.</span></span>

<span data-ttu-id="15d9a-194">Most ismételje meg a Surveys alkalmazás ugyanezekkel a lépésekkel kivéve ne adjon hozzá egy bejegyzés `knownClientApplications`.</span><span class="sxs-lookup"><span data-stu-id="15d9a-194">Now repeat the same steps for the Surveys app, except do not add an entry for `knownClientApplications`.</span></span> <span data-ttu-id="15d9a-195">Használja ugyanazt a szerepkör-definíciók, de új GUID azonosítóját hozza létre.</span><span class="sxs-lookup"><span data-stu-id="15d9a-195">Use the same role definitions, but generate new GUIDs for the IDs.</span></span>

## <a name="create-a-new-redis-cache-instance"></a><span data-ttu-id="15d9a-196">Új Redis Cache-példány létrehozása</span><span class="sxs-lookup"><span data-stu-id="15d9a-196">Create a new Redis Cache instance</span></span>

<span data-ttu-id="15d9a-197">A Surveys alkalmazás a Redis cache OAuth 2 hozzáférési jogkivonatok használja.</span><span class="sxs-lookup"><span data-stu-id="15d9a-197">The Surveys application uses Redis to cache OAuth 2 access tokens.</span></span> <span data-ttu-id="15d9a-198">A gyorsítótár létrehozásához:</span><span class="sxs-lookup"><span data-stu-id="15d9a-198">To create the cache:</span></span>

1. <span data-ttu-id="15d9a-199">Lépjen a [az Azure Portal](https://portal.azure.com) kattintson **+ erőforrás létrehozása** > **adatbázisok** > **Redis Cache**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-199">Go to [Azure Portal](https://portal.azure.com) and click **+ Create a Resource** > **Databases** > **Redis Cache**.</span></span>

2. <span data-ttu-id="15d9a-200">Adja meg a szükséges információkat, beleértve a DNS-név, erőforráscsoport, helyét, és a tarifacsomag.</span><span class="sxs-lookup"><span data-stu-id="15d9a-200">Fill in the required information, including DNS name, resource group, location, and pricing tier.</span></span> <span data-ttu-id="15d9a-201">Hozzon létre egy új erőforráscsoportot, vagy használjon egy meglévő erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="15d9a-201">You can create a new resource group or use an existing resource group.</span></span>

3. <span data-ttu-id="15d9a-202">Kattintson a **Create** (Létrehozás) gombra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-202">Click **Create**.</span></span>

4. <span data-ttu-id="15d9a-203">A Redis gyorsítótár létrehozása után keresse meg az erőforrást a portálon.</span><span class="sxs-lookup"><span data-stu-id="15d9a-203">After the Redis cache is created, navigate to the resource in the portal.</span></span>

5. <span data-ttu-id="15d9a-204">Kattintson a **hozzáférési kulcsok** , és másolja az elsődleges kulcsot.</span><span class="sxs-lookup"><span data-stu-id="15d9a-204">Click **Access keys** and copy the primary key.</span></span>

<span data-ttu-id="15d9a-205">Egy Redis cache létrehozásával kapcsolatos további információkért lásd: [How to Azure Redis Cache használata](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache).</span><span class="sxs-lookup"><span data-stu-id="15d9a-205">For more information about creating a Redis cache, see [How to Use Azure Redis Cache](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache).</span></span>

## <a name="set-application-secrets"></a><span data-ttu-id="15d9a-206">Titkos alkalmazáskulcsok beállítása</span><span class="sxs-lookup"><span data-stu-id="15d9a-206">Set application secrets</span></span>

1. <span data-ttu-id="15d9a-207">Nyissa meg a Tailspin.Surveys megoldást a Visual Studióban.</span><span class="sxs-lookup"><span data-stu-id="15d9a-207">Open the Tailspin.Surveys solution in Visual Studio.</span></span>

2. <span data-ttu-id="15d9a-208">A Megoldáskezelőben kattintson a jobb gombbal a Tailspin.Surveys.Web projektet, és válassza ki **felhasználói titkok kezelése**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-208">In Solution Explorer, right-click the Tailspin.Surveys.Web project and select **Manage User Secrets**.</span></span>

3. <span data-ttu-id="15d9a-209">A secrets.json fájlban illessze be a következőket:</span><span class="sxs-lookup"><span data-stu-id="15d9a-209">In the secrets.json file, paste in the following:</span></span>

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

    <span data-ttu-id="15d9a-210">Cserélje le a csúcsos zárójelpárban van, a következő megjelenített elemek:</span><span class="sxs-lookup"><span data-stu-id="15d9a-210">Replace the items shown in angle brackets, as follows:</span></span>

    - <span data-ttu-id="15d9a-211">`AzureAd:ClientId`: Az a Surveys alkalmazás azonosítója.</span><span class="sxs-lookup"><span data-stu-id="15d9a-211">`AzureAd:ClientId`: The application ID of the Surveys app.</span></span>
    - <span data-ttu-id="15d9a-212">`AzureAd:ClientSecret`: Az Azure ad-ben a Surveys alkalmazás regisztrációja során létrehozott kulcs.</span><span class="sxs-lookup"><span data-stu-id="15d9a-212">`AzureAd:ClientSecret`: The key that you generated when you registered the Surveys application in Azure AD.</span></span>
    - <span data-ttu-id="15d9a-213">`AzureAd:WebApiResourceId`: Az Alkalmazásazonosító URI-t, hogy a megadott Azure AD-ben a Surveys.WebAPI alkalmazás létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="15d9a-213">`AzureAd:WebApiResourceId`: The App ID URI that you specified when you created the Surveys.WebAPI application in Azure AD.</span></span> <span data-ttu-id="15d9a-214">Az űrlap kell rendelkeznie `https://<directory>.onmicrosoft.com/surveys.webapi`</span><span class="sxs-lookup"><span data-stu-id="15d9a-214">It should have the form `https://<directory>.onmicrosoft.com/surveys.webapi`</span></span>
    - <span data-ttu-id="15d9a-215">`Redis:Configuration`: Ez a karakterlánc, a DNS-nevét a Redis cache és az elsődleges elérési kulcs létrehozása.</span><span class="sxs-lookup"><span data-stu-id="15d9a-215">`Redis:Configuration`: Build this string from the DNS name of the Redis cache and the primary access key.</span></span> <span data-ttu-id="15d9a-216">For example, "tailspin.redis.cache.windows.net,password=2h5tBxxx,ssl=true".</span><span class="sxs-lookup"><span data-stu-id="15d9a-216">For example, "tailspin.redis.cache.windows.net,password=2h5tBxxx,ssl=true".</span></span>

4. <span data-ttu-id="15d9a-217">Mentse a frissített secrets.json fájlt.</span><span class="sxs-lookup"><span data-stu-id="15d9a-217">Save the updated secrets.json file.</span></span>

5. <span data-ttu-id="15d9a-218">Ismételje meg ezeket a lépéseket a Tailspin.Surveys.WebAPI projekt, de secrets.json illessze be a következő.</span><span class="sxs-lookup"><span data-stu-id="15d9a-218">Repeat these steps for the Tailspin.Surveys.WebAPI project, but paste the following into secrets.json.</span></span> <span data-ttu-id="15d9a-219">Ahogy korábban is cserélje le a csúcsos zárójelpárban van, a cikkeket.</span><span class="sxs-lookup"><span data-stu-id="15d9a-219">Replace the items in angle brackets, as before.</span></span>

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

## <a name="initialize-the-database"></a><span data-ttu-id="15d9a-220">Az adatbázis inicializálása</span><span class="sxs-lookup"><span data-stu-id="15d9a-220">Initialize the database</span></span>

<span data-ttu-id="15d9a-221">Ebben a lépésben használandó entitás-keretrendszer 7 LocalDB használatakor egy helyi SQL-adatbázis létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="15d9a-221">In this step, you will use Entity Framework 7 to create a local SQL database, using LocalDB.</span></span>

1. <span data-ttu-id="15d9a-222">Nyisson meg egy parancsablakot</span><span class="sxs-lookup"><span data-stu-id="15d9a-222">Open a command window</span></span>

2. <span data-ttu-id="15d9a-223">Navigate to the Tailspin.Surveys.Data project.</span><span class="sxs-lookup"><span data-stu-id="15d9a-223">Navigate to the Tailspin.Surveys.Data project.</span></span>

3. <span data-ttu-id="15d9a-224">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="15d9a-224">Run the following command:</span></span>

    ```bat
    dotnet ef database update --startup-project ..\Tailspin.Surveys.Web
    ```

## <a name="run-the-application"></a><span data-ttu-id="15d9a-225">Az alkalmazás futtatása</span><span class="sxs-lookup"><span data-stu-id="15d9a-225">Run the application</span></span>

<span data-ttu-id="15d9a-226">Futtassa az alkalmazást, indítsa el a Tailspin.Surveys.Web és a Tailspin.Surveys.WebAPI projektek.</span><span class="sxs-lookup"><span data-stu-id="15d9a-226">To run the application, start both the Tailspin.Surveys.Web and Tailspin.Surveys.WebAPI projects.</span></span>

<span data-ttu-id="15d9a-227">Visual Studio segítségével mindkét projekt futtatásához automatikusan F5 billentyűt, a következőképpen állíthatja be:</span><span class="sxs-lookup"><span data-stu-id="15d9a-227">You can set Visual Studio to run both projects automatically on F5, as follows:</span></span>

1. <span data-ttu-id="15d9a-228">A Megoldáskezelőben kattintson a jobb gombbal a megoldás, és kattintson a **indítási projektek beállítása**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-228">In Solution Explorer, right-click the solution and click **Set Startup Projects**.</span></span>
2. <span data-ttu-id="15d9a-229">Válassza ki **több kezdőprojekt**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-229">Select **Multiple startup projects**.</span></span>
3. <span data-ttu-id="15d9a-230">Állítsa be **művelet** = **Start** Tailspin.Surveys.Web és Tailspin.Surveys.WebAPI projektek.</span><span class="sxs-lookup"><span data-stu-id="15d9a-230">Set **Action** = **Start** for the Tailspin.Surveys.Web and Tailspin.Surveys.WebAPI projects.</span></span>

## <a name="sign-up-a-new-tenant"></a><span data-ttu-id="15d9a-231">Regisztráljon egy új bérlőt</span><span class="sxs-lookup"><span data-stu-id="15d9a-231">Sign up a new tenant</span></span>

<span data-ttu-id="15d9a-232">Az alkalmazás indításakor nem jelentkezett be, így az üdvözlő oldal jelenik meg:</span><span class="sxs-lookup"><span data-stu-id="15d9a-232">When the application starts, you are not signed in, so you see the welcome page:</span></span>

![Kezdőlap](./images/running-the-app/screenshot1.png)

<span data-ttu-id="15d9a-234">Regisztráció a szervezet:</span><span class="sxs-lookup"><span data-stu-id="15d9a-234">To sign up an organization:</span></span>

1. <span data-ttu-id="15d9a-235">Kattintson a **regisztrálni a vállalat a Tailspin**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-235">Click **Enroll your company in Tailspin**.</span></span>
2. <span data-ttu-id="15d9a-236">Jelentkezzen be az Azure AD-címtár, amely a szervezet a Surveys alkalmazással jelöli.</span><span class="sxs-lookup"><span data-stu-id="15d9a-236">Sign in to the Azure AD directory that represents the organization using the Surveys app.</span></span> <span data-ttu-id="15d9a-237">Rendszergazda felhasználóként jelentkezzen be.</span><span class="sxs-lookup"><span data-stu-id="15d9a-237">You must sign in as an admin user.</span></span>
3. <span data-ttu-id="15d9a-238">Fogadja el a beleegyezést kérő üzenetet.</span><span class="sxs-lookup"><span data-stu-id="15d9a-238">Accept the consent prompt.</span></span>

<span data-ttu-id="15d9a-239">Az alkalmazás a bérlő regisztrálja, és ezután jelentkezteti ki. Az alkalmazás kijelentkezik, mert szüksége lesz az alkalmazás-szerepkörök beállítása az Azure AD-ben az alkalmazás használata előtt.</span><span class="sxs-lookup"><span data-stu-id="15d9a-239">The application registers the tenant, and then signs you out. The app signs you out because you need to set up the application roles in Azure AD, before using the application.</span></span>

![Regisztráció után](./images/running-the-app/screenshot2.png)

## <a name="assign-application-roles"></a><span data-ttu-id="15d9a-241">Alkalmazás-szerepkörök hozzárendelése</span><span class="sxs-lookup"><span data-stu-id="15d9a-241">Assign application roles</span></span>

<span data-ttu-id="15d9a-242">Amikor egy bérlő regisztrál, a bérlői rendszergazda AD hozzá kell rendelnie alkalmazás-szerepkörök a felhasználók számára.</span><span class="sxs-lookup"><span data-stu-id="15d9a-242">When a tenant signs up, an AD admin for the tenant must assign application roles to users.</span></span>

1. <span data-ttu-id="15d9a-243">Az a [az Azure portal][portal], váltson át az Azure AD-címtár, amellyel a Surveys alkalmazás regisztrálhat.</span><span class="sxs-lookup"><span data-stu-id="15d9a-243">In the [Azure portal][portal], switch to the Azure AD directory that you used to sign up for the Surveys app.</span></span>

2. <span data-ttu-id="15d9a-244">A bal oldali navigációs ablaktáblán válassza ki a **Azure Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-244">In the left-hand navigation pane, choose **Azure Active Directory**.</span></span>

3. <span data-ttu-id="15d9a-245">Kattintson a **vállalati alkalmazások** > **minden alkalmazás**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-245">Click **Enterprise applications** > **All applications**.</span></span> <span data-ttu-id="15d9a-246">A portálon megjelennek `Survey` és `Survey.WebAPI`.</span><span class="sxs-lookup"><span data-stu-id="15d9a-246">The portal will list `Survey` and `Survey.WebAPI`.</span></span> <span data-ttu-id="15d9a-247">Ha nem, akkor győződjön meg arról, hogy elvégezte-e a regisztrációs folyamat.</span><span class="sxs-lookup"><span data-stu-id="15d9a-247">If not, make sure that you completed the sign up process.</span></span>

4. <span data-ttu-id="15d9a-248">Kattintson a Surveys alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="15d9a-248">Click on the Surveys application.</span></span>

5. <span data-ttu-id="15d9a-249">Kattintson a **felhasználók és csoportok**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-249">Click **Users and Groups**.</span></span>

6. <span data-ttu-id="15d9a-250">Kattintson a **Felhasználó hozzáadása** parancsra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-250">Click **Add user**.</span></span>

7. <span data-ttu-id="15d9a-251">Ha rendelkezik Azure AD Premium szolgáltatással, kattintson a **felhasználók és csoportok**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-251">If you have Azure AD Premium, click **Users and groups**.</span></span> <span data-ttu-id="15d9a-252">Ellenkező esetben kattintson a **felhasználók**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-252">Otherwise, click **Users**.</span></span> <span data-ttu-id="15d9a-253">(A prémium szintű Azure AD szerepkör hozzárendelése egy csoporthoz szükséges.)</span><span class="sxs-lookup"><span data-stu-id="15d9a-253">(Assigning a role to a group requires Azure AD Premium.)</span></span>

8. <span data-ttu-id="15d9a-254">Válassza ki egy vagy több felhasználót, és kattintson a **kiválasztása**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-254">Select one or more users and click **Select**.</span></span>

    ![Felhasználó vagy csoport kiválasztása](./images/running-the-app/select-user-or-group.png)

9. <span data-ttu-id="15d9a-256">Válassza ki a szerepkört, és kattintson a **kiválasztása**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-256">Select the role and click **Select**.</span></span>

    ![Felhasználó vagy csoport kiválasztása](./images/running-the-app/select-role.png)

10. <span data-ttu-id="15d9a-258">Kattintson a **Hozzárendelés** gombra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-258">Click **Assign**.</span></span>

<span data-ttu-id="15d9a-259">Ismételje meg a ugyanazokat a lépéseket, ha Survey.WebAPI alkalmazásához szerepköröket.</span><span class="sxs-lookup"><span data-stu-id="15d9a-259">Repeat the same steps to assign roles for the Survey.WebAPI application.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="15d9a-260">A felhasználó a felmérési és a Survey.WebAPI ugyanezeket a szerepköröket mindig rendelkeznie kell.</span><span class="sxs-lookup"><span data-stu-id="15d9a-260">A user should always have the same roles in both Survey and Survey.WebAPI.</span></span> <span data-ttu-id="15d9a-261">Ellenkező esetben a felhasználói jogosultak lesznek inkonzisztens, amely a webes API a 403-as (tiltott) hibákhoz vezethetnek.</span><span class="sxs-lookup"><span data-stu-id="15d9a-261">Otherwise, the user will have inconsistent permissions, which may lead to 403 (Forbidden) errors from the Web API.</span></span>

<span data-ttu-id="15d9a-262">Most lépjen vissza az alkalmazást, és jelentkezzen be újra.</span><span class="sxs-lookup"><span data-stu-id="15d9a-262">Now go back to the app and sign in again.</span></span> <span data-ttu-id="15d9a-263">Kattintson a **saját felmérések**.</span><span class="sxs-lookup"><span data-stu-id="15d9a-263">Click **My Surveys**.</span></span> <span data-ttu-id="15d9a-264">Ha a felhasználói szerepkör van rendelve, a SurveyAdmin vagy SurveyCreator, látni fogja a **felmérés létrehozása** gomb, amely azt jelzi, hogy a felhasználók számára hozzon létre egy új felmérésre.</span><span class="sxs-lookup"><span data-stu-id="15d9a-264">If the user is assigned to the SurveyAdmin or SurveyCreator role, you will see a **Create Survey** button, indicating that the user has permissions to create a new survey.</span></span>

![A felmérések](./images/running-the-app/screenshot3.png)

<!-- links -->

[portal]: https://portal.azure.com
[VS2017]: https://www.visualstudio.com/vs/
