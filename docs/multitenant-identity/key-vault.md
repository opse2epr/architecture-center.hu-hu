---
title: Key Vault használata az alkalmazás titkainak védelmére
description: A Key Vault szolgáltatás használata az alkalmazás titkainak tárolásához
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
ms.openlocfilehash: a47b8835df1c970ac7c50af78aae73116d6a12b4
ms.sourcegitcommit: d59e2631fb08665bc30f6b65bfc7e1b75935cbd5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/05/2018
ms.locfileid: "51021950"
---
# <a name="use-azure-key-vault-to-protect-application-secrets"></a><span data-ttu-id="129d3-103">Azure Key Vault használata az alkalmazás titkainak védelmére</span><span class="sxs-lookup"><span data-stu-id="129d3-103">Use Azure Key Vault to protect application secrets</span></span>

<span data-ttu-id="129d3-104">[![GitHub](../_images/github.png) Mintakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="129d3-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="129d3-105">Célszerű a gyakori, hogy olyan alkalmazásbeállításokat, amelyek kis-és nagybetűket, és védeni kell, például:</span><span class="sxs-lookup"><span data-stu-id="129d3-105">It's common to have application settings that are sensitive and must be protected, such as:</span></span>

* <span data-ttu-id="129d3-106">Adatbázisok kapcsolati sztringjei</span><span class="sxs-lookup"><span data-stu-id="129d3-106">Database connection strings</span></span>
* <span data-ttu-id="129d3-107">Jelszavak</span><span class="sxs-lookup"><span data-stu-id="129d3-107">Passwords</span></span>
* <span data-ttu-id="129d3-108">Titkosítási kulcsok</span><span class="sxs-lookup"><span data-stu-id="129d3-108">Cryptographic keys</span></span>

<span data-ttu-id="129d3-109">Bevált biztonsági gyakorlat verziókövetés titkos adatokat soha nem tárolja.</span><span class="sxs-lookup"><span data-stu-id="129d3-109">As a security best practice, you should never store these secrets in source control.</span></span> <span data-ttu-id="129d3-110">Túl egyszerű, hogy nyilvánosságra kerüljenek &mdash; akkor is, ha a forráskód adattára privát.</span><span class="sxs-lookup"><span data-stu-id="129d3-110">It's too easy for them to leak &mdash; even if your source code repository is private.</span></span> <span data-ttu-id="129d3-111">És azt nem szinte megakadályozza a titkos kulcsok az általános nyilvános.</span><span class="sxs-lookup"><span data-stu-id="129d3-111">And it's not just about keeping secrets from the general public.</span></span> <span data-ttu-id="129d3-112">A nagyobb projektek mely fejlesztők korlátozni lehet, hogy szeretné, és operátorok hozzáférnek a termelési titkos kulcsait.</span><span class="sxs-lookup"><span data-stu-id="129d3-112">On larger projects, you might want to restrict which developers and operators can access the production secrets.</span></span> <span data-ttu-id="129d3-113">(A tesztelési vagy fejlesztési környezetek beállítások eltérőek.)</span><span class="sxs-lookup"><span data-stu-id="129d3-113">(Settings for test or development environments are different.)</span></span>

<span data-ttu-id="129d3-114">Egy biztonságosabb megoldás, ha a titkos adatokat tárolni [Azure Key Vault][KeyVault].</span><span class="sxs-lookup"><span data-stu-id="129d3-114">A more secure option is to store these secrets in [Azure Key Vault][KeyVault].</span></span> <span data-ttu-id="129d3-115">A Key Vault egy olyan felhőben üzemeltetett szolgáltatás titkosítási kulcsok és egyéb titkok kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="129d3-115">Key Vault is a cloud-hosted service for managing cryptographic keys and other secrets.</span></span> <span data-ttu-id="129d3-116">Ez a cikk bemutatja, hogyan tárolja a konfigurációs beállításokat az alkalmazás a Key Vault használatával.</span><span class="sxs-lookup"><span data-stu-id="129d3-116">This article shows how to use Key Vault to store configuration settings for your app.</span></span>

<span data-ttu-id="129d3-117">Az a [Tailspin Surveys] [ Surveys] alkalmazás, a következő beállításokat is titkos:</span><span class="sxs-lookup"><span data-stu-id="129d3-117">In the [Tailspin Surveys][Surveys] application, the following settings are secret:</span></span>

* <span data-ttu-id="129d3-118">Az adatbázis-kapcsolati karakterláncot.</span><span class="sxs-lookup"><span data-stu-id="129d3-118">The database connection string.</span></span>
* <span data-ttu-id="129d3-119">A Redis kapcsolati karakterlánc.</span><span class="sxs-lookup"><span data-stu-id="129d3-119">The Redis connection string.</span></span>
* <span data-ttu-id="129d3-120">A titkos ügyfélkulcsot a webes alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="129d3-120">The client secret for the web application.</span></span>

<span data-ttu-id="129d3-121">A Surveys alkalmazás konfigurációs beállítások tölt be a következő helyeken:</span><span class="sxs-lookup"><span data-stu-id="129d3-121">The Surveys application loads configuration settings from the following places:</span></span>

* <span data-ttu-id="129d3-122">Az appsettings.json fájl</span><span class="sxs-lookup"><span data-stu-id="129d3-122">The appsettings.json file</span></span>
* <span data-ttu-id="129d3-123">A [felhasználói titkos kódok store] [ user-secrets] (fejlesztési környezet; csak tesztelési célra)</span><span class="sxs-lookup"><span data-stu-id="129d3-123">The [user secrets store][user-secrets] (development environment only; for testing)</span></span>
* <span data-ttu-id="129d3-124">Az üzemeltetési környezet (alkalmazás beállításai az Azure web apps)</span><span class="sxs-lookup"><span data-stu-id="129d3-124">The hosting environment (app settings in Azure web apps)</span></span>
* <span data-ttu-id="129d3-125">A Key Vault (Ha engedélyezve van)</span><span class="sxs-lookup"><span data-stu-id="129d3-125">Key Vault (when enabled)</span></span>

<span data-ttu-id="129d3-126">Minden egyes ezeket a felülbírálásokat az előzőt, így a Key vaultban tárolt beállítások elsőbbséget.</span><span class="sxs-lookup"><span data-stu-id="129d3-126">Each of these overrides the previous one, so any settings stored in Key Vault take precedence.</span></span>

> [!NOTE]
> <span data-ttu-id="129d3-127">A Key Vault konfigurációszolgáltató alapértelmezés szerint le van tiltva.</span><span class="sxs-lookup"><span data-stu-id="129d3-127">By default, the Key Vault configuration provider is disabled.</span></span> <span data-ttu-id="129d3-128">Nincs szükség az alkalmazás helyi futtatásához.</span><span class="sxs-lookup"><span data-stu-id="129d3-128">It's not needed for running the application locally.</span></span> <span data-ttu-id="129d3-129">Éles környezetben azt szeretné engedélyezni.</span><span class="sxs-lookup"><span data-stu-id="129d3-129">You would enable it in a production deployment.</span></span>

<span data-ttu-id="129d3-130">Indításkor az alkalmazás minden regisztrált konfigurációszolgáltató beállításokat olvas, és feltölti egy szigorú típusmegadású objektum használja őket.</span><span class="sxs-lookup"><span data-stu-id="129d3-130">At startup, the application reads settings from every registered configuration provider, and uses them to populate a strongly typed options object.</span></span> <span data-ttu-id="129d3-131">További információkért lásd: [beállítások használatával és konfigurációs objektumok][options].</span><span class="sxs-lookup"><span data-stu-id="129d3-131">For more information, see [Using Options and configuration objects][options].</span></span>

## <a name="setting-up-key-vault-in-the-surveys-app"></a><span data-ttu-id="129d3-132">A Surveys alkalmazás Key Vault beállítása</span><span class="sxs-lookup"><span data-stu-id="129d3-132">Setting up Key Vault in the Surveys app</span></span>
<span data-ttu-id="129d3-133">Előfeltételek:</span><span class="sxs-lookup"><span data-stu-id="129d3-133">Prerequisites:</span></span>

* <span data-ttu-id="129d3-134">Telepítse a [Azure Resource Manager parancsmagjainak][azure-rm-cmdlets].</span><span class="sxs-lookup"><span data-stu-id="129d3-134">Install the [Azure Resource Manager Cmdlets][azure-rm-cmdlets].</span></span>
* <span data-ttu-id="129d3-135">Konfigurálja a Surveys alkalmazás leírtak szerint [a Surveys alkalmazás futtatása][readme].</span><span class="sxs-lookup"><span data-stu-id="129d3-135">Configure the Surveys application as described in [Run the Surveys application][readme].</span></span>

<span data-ttu-id="129d3-136">Magas szintű lépéseket:</span><span class="sxs-lookup"><span data-stu-id="129d3-136">High-level steps:</span></span>

1. <span data-ttu-id="129d3-137">Állítsa be egy rendszergazdai felhasználót a bérlőben.</span><span class="sxs-lookup"><span data-stu-id="129d3-137">Set up an admin user in the tenant.</span></span>
2. <span data-ttu-id="129d3-138">Állítsa be az ügyféltanúsítványt.</span><span class="sxs-lookup"><span data-stu-id="129d3-138">Set up a client certificate.</span></span>
3. <span data-ttu-id="129d3-139">Kulcstartó létrehozása.</span><span class="sxs-lookup"><span data-stu-id="129d3-139">Create a key vault.</span></span>
4. <span data-ttu-id="129d3-140">Konfigurációs beállítások hozzáadása a kulcstartóhoz.</span><span class="sxs-lookup"><span data-stu-id="129d3-140">Add configuration settings to your key vault.</span></span>
5. <span data-ttu-id="129d3-141">Állítsa vissza a kódot, amely lehetővé teszi, hogy a key vaultban.</span><span class="sxs-lookup"><span data-stu-id="129d3-141">Uncomment the code that enables key vault.</span></span>
6. <span data-ttu-id="129d3-142">Az alkalmazás felhasználói titkos kódok frissítése.</span><span class="sxs-lookup"><span data-stu-id="129d3-142">Update the application's user secrets.</span></span>

### <a name="set-up-an-admin-user"></a><span data-ttu-id="129d3-143">Egy rendszergazda felhasználó beállítása</span><span class="sxs-lookup"><span data-stu-id="129d3-143">Set up an admin user</span></span>
> [!NOTE]
> <span data-ttu-id="129d3-144">Hozzon létre egy kulcstartót, olyan fiók, amely az Azure-előfizetés kezelése céljából kell használnia.</span><span class="sxs-lookup"><span data-stu-id="129d3-144">To create a key vault, you must use an account which can manage your Azure subscription.</span></span> <span data-ttu-id="129d3-145">Is minden olyan alkalmazás, hogy olvassa el a key vaultban tárolt elérjék regisztrálni kell, hogy a fiók ugyanabban a bérlőben.</span><span class="sxs-lookup"><span data-stu-id="129d3-145">Also, any application that you authorize to read from the key vault must be registered in the same tenant as that account.</span></span>
> 
> 

<span data-ttu-id="129d3-146">Ebben a lépésben teheti meg arról, hogy hozhat létre egy kulcstartót, amíg a bérlő felhasználóként jelentkezett be, a Surveys alkalmazás regisztrálva van-e.</span><span class="sxs-lookup"><span data-stu-id="129d3-146">In this step, you will make sure that you can create a key vault while signed in as a user from the tenant where the Surveys app is registered.</span></span>

<span data-ttu-id="129d3-147">Hozzon létre egy rendszergazda felhasználó belül az Azure AD-bérlővel, ahol a Surveys alkalmazás regisztrálva van-e.</span><span class="sxs-lookup"><span data-stu-id="129d3-147">Create an administrator user within the Azure AD tenant where the Surveys application is registered.</span></span>

1. <span data-ttu-id="129d3-148">Jelentkezzen be a [az Azure portal][azure-portal].</span><span class="sxs-lookup"><span data-stu-id="129d3-148">Log into the [Azure portal][azure-portal].</span></span>
2. <span data-ttu-id="129d3-149">Válassza ki az Azure AD-bérlővel, ahol az alkalmazás regisztrálva lesz.</span><span class="sxs-lookup"><span data-stu-id="129d3-149">Select the Azure AD tenant where your application is registered.</span></span>
3. <span data-ttu-id="129d3-150">Kattintson a **további szolgáltatás** > **biztonság + IDENTITÁS** > **Azure Active Directory** > **felhasználók és csoportok**   >  **Minden felhasználó**.</span><span class="sxs-lookup"><span data-stu-id="129d3-150">Click **More service** > **SECURITY + IDENTITY** > **Azure Active Directory** > **User and groups** > **All users**.</span></span>
4. <span data-ttu-id="129d3-151">Kattintson a portál tetején **új felhasználó**.</span><span class="sxs-lookup"><span data-stu-id="129d3-151">At the top of the portal, click **New user**.</span></span>
5. <span data-ttu-id="129d3-152">Töltse ki a mezőket, és rendelje hozzá a felhasználót a **globális rendszergazdai** címtárbeli szerepkör.</span><span class="sxs-lookup"><span data-stu-id="129d3-152">Fill in the fields and assign the user to the **Global administrator** directory role.</span></span>
6. <span data-ttu-id="129d3-153">Kattintson a **Create** (Létrehozás) gombra.</span><span class="sxs-lookup"><span data-stu-id="129d3-153">Click **Create**.</span></span>

![Globális rendszergazda](./images/running-the-app/global-admin-user.png)

<span data-ttu-id="129d3-155">Most rendelje hozzá a felhasználó az előfizetés tulajdonosaként.</span><span class="sxs-lookup"><span data-stu-id="129d3-155">Now assign this user as the subscription owner.</span></span>

1. <span data-ttu-id="129d3-156">A központ menüben válassza ki a **előfizetések**.</span><span class="sxs-lookup"><span data-stu-id="129d3-156">On the Hub menu, select **Subscriptions**.</span></span>

    ![](./images/running-the-app/subscriptions.png)

2. <span data-ttu-id="129d3-157">A rendszergazda eléréséhez használni kívánt előfizetés kiválasztásához.</span><span class="sxs-lookup"><span data-stu-id="129d3-157">Select the subscription that you want the administrator to access.</span></span>
3. <span data-ttu-id="129d3-158">Az előfizetési panelen válassza ki a **hozzáférés-vezérlés (IAM)**.</span><span class="sxs-lookup"><span data-stu-id="129d3-158">In the subscription blade, select **Access control (IAM)**.</span></span>
4. <span data-ttu-id="129d3-159">Kattintson a **Hozzáadás** parancsra.</span><span class="sxs-lookup"><span data-stu-id="129d3-159">Click **Add**.</span></span>
4. <span data-ttu-id="129d3-160">A **szerepkör**válassza **tulajdonosa**.</span><span class="sxs-lookup"><span data-stu-id="129d3-160">Under **Role**, select **Owner**.</span></span>
5. <span data-ttu-id="129d3-161">Írja be a tulajdonosként hozzáadni kívánt felhasználó e-mail-címét.</span><span class="sxs-lookup"><span data-stu-id="129d3-161">Type the email address of the user you want to add as owner.</span></span>
6. <span data-ttu-id="129d3-162">Válassza ki a felhasználót, és kattintson a **mentése**.</span><span class="sxs-lookup"><span data-stu-id="129d3-162">Select the user and click **Save**.</span></span>

### <a name="set-up-a-client-certificate"></a><span data-ttu-id="129d3-163">Ügyfél-tanúsítvány beállítása</span><span class="sxs-lookup"><span data-stu-id="129d3-163">Set up a client certificate</span></span>
1. <span data-ttu-id="129d3-164">A PowerShell-parancsprogrammal [/Scripts/Setup-KeyVault.ps1] [ Setup-KeyVault] módon:</span><span class="sxs-lookup"><span data-stu-id="129d3-164">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    <span data-ttu-id="129d3-165">Az a `Subject` paramétert, tetszőleges nevet, például a "surveysapp" adja meg.</span><span class="sxs-lookup"><span data-stu-id="129d3-165">For the `Subject` parameter, enter any name, such as "surveysapp".</span></span> <span data-ttu-id="129d3-166">A szkript létrehoz egy önaláírt tanúsítványt, és tárolja azokat az "aktuális User/Personal" tanúsítványtárolójában.</span><span class="sxs-lookup"><span data-stu-id="129d3-166">The script generates a self-signed certificate and stores it in the "Current User/Personal" certificate store.</span></span> <span data-ttu-id="129d3-167">A parancsprogram kimenete egy JSON-töredék.</span><span class="sxs-lookup"><span data-stu-id="129d3-167">The output from the script is a JSON fragment.</span></span> <span data-ttu-id="129d3-168">Másolja ezt az értéket.</span><span class="sxs-lookup"><span data-stu-id="129d3-168">Copy this value.</span></span>

2. <span data-ttu-id="129d3-169">Az a [az Azure portal][azure-portal], lépjen abba a könyvtárba, ahol a Surveys alkalmazás regisztrálva van, a portál jobb felső sarkában a fiók kiválasztásával.</span><span class="sxs-lookup"><span data-stu-id="129d3-169">In the [Azure portal][azure-portal], switch to the directory where the Surveys application is registered, by selecting your account in the top right corner of the portal.</span></span>

3. <span data-ttu-id="129d3-170">Válassza ki **az Azure Active Directory** > **Alkalmazásregisztrációk** > felmérések</span><span class="sxs-lookup"><span data-stu-id="129d3-170">Select **Azure Active Directory** > **App Registrations** > Surveys</span></span>

4.  <span data-ttu-id="129d3-171">Kattintson a **Manifest** , majd **szerkesztése**.</span><span class="sxs-lookup"><span data-stu-id="129d3-171">Click **Manifest** and then **Edit**.</span></span>

5.  <span data-ttu-id="129d3-172">Illessze be a szkript kimenetében a `keyCredentials` tulajdonság.</span><span class="sxs-lookup"><span data-stu-id="129d3-172">Paste the output from the script into the `keyCredentials` property.</span></span> <span data-ttu-id="129d3-173">Ennek a következőképpen kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="129d3-173">It should look similar to the following:</span></span>
        
    ```json
    "keyCredentials": [
        {
        "type": "AsymmetricX509Cert",
        "usage": "Verify",
        "keyId": "29d4f7db-0539-455e-b708-....",
        "customKeyIdentifier": "ZEPpP/+KJe2fVDBNaPNOTDoJMac=",
        "value": "MIIDAjCCAeqgAwIBAgIQFxeRiU59eL.....
        }
    ],
    ```          

6. <span data-ttu-id="129d3-174">Kattintson a **Save** (Mentés) gombra.</span><span class="sxs-lookup"><span data-stu-id="129d3-174">Click **Save**.</span></span>  

7. <span data-ttu-id="129d3-175">Ismételje meg a 3-6 JSON ugyanarra a töredékre hozzáadása az alkalmazásjegyzékben, a webes API (Surveys.WebAPI).</span><span class="sxs-lookup"><span data-stu-id="129d3-175">Repeat steps 3-6 to add the same JSON fragment to the application manifest of the web API (Surveys.WebAPI).</span></span>

8. <span data-ttu-id="129d3-176">A PowerShell-ablakban futtassa a következő parancsot a tanúsítvány ujjlenyomatának beolvasása.</span><span class="sxs-lookup"><span data-stu-id="129d3-176">From the PowerShell window, run the following command to get the thumbprint of the certificate.</span></span>
   
    ```
    certutil -store -user my [subject]
    ```
    
    <span data-ttu-id="129d3-177">A `[subject]`, használja az értéket, a PowerShell-parancsfájl a megadott tulajdonos.</span><span class="sxs-lookup"><span data-stu-id="129d3-177">For `[subject]`, use the value that you specified for Subject in the PowerShell script.</span></span> <span data-ttu-id="129d3-178">Az ujjlenyomat "Tanúsítvány SHA1" területen látható.</span><span class="sxs-lookup"><span data-stu-id="129d3-178">The thumbprint is listed under "Cert Hash(sha1)".</span></span> <span data-ttu-id="129d3-179">Másolja ezt az értéket.</span><span class="sxs-lookup"><span data-stu-id="129d3-179">Copy this value.</span></span> <span data-ttu-id="129d3-180">Az ujjlenyomat később fogja használni.</span><span class="sxs-lookup"><span data-stu-id="129d3-180">You will use the thumbprint later.</span></span>

### <a name="create-a-key-vault"></a><span data-ttu-id="129d3-181">Kulcstartó létrehozása</span><span class="sxs-lookup"><span data-stu-id="129d3-181">Create a key vault</span></span>
1. <span data-ttu-id="129d3-182">A PowerShell-parancsprogrammal [/Scripts/Setup-KeyVault.ps1] [ Setup-KeyVault] módon:</span><span class="sxs-lookup"><span data-stu-id="129d3-182">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```
   
    <span data-ttu-id="129d3-183">Amikor a rendszer kéri a hitelesítő adatokat, jelentkezzen be az Azure AD-felhasználóként előzőleg létrehozott jelszóra.</span><span class="sxs-lookup"><span data-stu-id="129d3-183">When prompted for credentials, sign in as the Azure AD user that you created earlier.</span></span> <span data-ttu-id="129d3-184">A szkript létrehoz egy új erőforráscsoportot, és a egy új kulcstartóba adott erőforráscsoporton belül.</span><span class="sxs-lookup"><span data-stu-id="129d3-184">The script creates a new resource group, and a new key vault within that resource group.</span></span> 
   
2. <span data-ttu-id="129d3-185">Futtassa újra a SetupKeyVault.ps az alábbiak szerint:</span><span class="sxs-lookup"><span data-stu-id="129d3-185">Run SetupKeyVault.ps again as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<Surveys app id>>", "<<Surveys.WebAPI app ID>>")
    ```
   
    <span data-ttu-id="129d3-186">Állítsa be a következő paraméter értékét:</span><span class="sxs-lookup"><span data-stu-id="129d3-186">Set the following parameter values:</span></span>
   
       * <span data-ttu-id="129d3-187">a Key vault name = a neve, mint a key vault az előző lépésben.</span><span class="sxs-lookup"><span data-stu-id="129d3-187">key vault name = The name that you gave the key vault in the previous step.</span></span>
       * <span data-ttu-id="129d3-188">Alkalmazásazonosító felmérések = az Alkalmazásazonosítót a felmérések webes alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="129d3-188">Surveys app ID = The application ID for the Surveys web application.</span></span>
       * <span data-ttu-id="129d3-189">Surveys.WebApi app ID = az Alkalmazásazonosítót a Surveys.WebAPI alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="129d3-189">Surveys.WebApi app ID = The application ID for the Surveys.WebAPI application.</span></span>
         
    <span data-ttu-id="129d3-190">Példa:</span><span class="sxs-lookup"><span data-stu-id="129d3-190">Example:</span></span>
     
    ```
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
    ```
    
    <span data-ttu-id="129d3-191">Ez a szkript engedélyezi a webalkalmazás és webes API titkos kódok lekérése a key vaultban.</span><span class="sxs-lookup"><span data-stu-id="129d3-191">This script authorizes the web app and web API to retrieve secrets from your key vault.</span></span> <span data-ttu-id="129d3-192">Lásd: [első lépései az Azure Key Vault](/azure/key-vault/key-vault-get-started/) további információt.</span><span class="sxs-lookup"><span data-stu-id="129d3-192">See [Get started with Azure Key Vault](/azure/key-vault/key-vault-get-started/) for more information.</span></span>

### <a name="add-configuration-settings-to-your-key-vault"></a><span data-ttu-id="129d3-193">Konfigurációs beállítások hozzáadása a kulcstartóhoz</span><span class="sxs-lookup"><span data-stu-id="129d3-193">Add configuration settings to your key vault</span></span>
1. <span data-ttu-id="129d3-194">SetupKeyVault.ps futtassa az alábbiak szerint:</span><span class="sxs-lookup"><span data-stu-id="129d3-194">Run SetupKeyVault.ps as follows::</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Redis--Configuration -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true" 
    ```
    <span data-ttu-id="129d3-195">ahol</span><span class="sxs-lookup"><span data-stu-id="129d3-195">where</span></span>
   
   * <span data-ttu-id="129d3-196">a Key vault name = a neve, mint a key vault az előző lépésben.</span><span class="sxs-lookup"><span data-stu-id="129d3-196">key vault name = The name that you gave the key vault in the previous step.</span></span>
   * <span data-ttu-id="129d3-197">Redis Cache a DNS-név = a Redis cache-példány a DNS-név.</span><span class="sxs-lookup"><span data-stu-id="129d3-197">Redis DNS name = The DNS name of your Redis cache instance.</span></span>
   * <span data-ttu-id="129d3-198">Redis Cache hozzáférési kulcs = a Redis cache-példány hozzáférési kulcsára.</span><span class="sxs-lookup"><span data-stu-id="129d3-198">Redis access key = The access key for your Redis cache instance.</span></span>
     
2. <span data-ttu-id="129d3-199">Ezen a ponton célszerű annak megállapítására, hogy sikerült menteni a titkos kulcsok a key vaulttal.</span><span class="sxs-lookup"><span data-stu-id="129d3-199">At this point, it's a good idea to test whether you successfully stored the secrets to key vault.</span></span> <span data-ttu-id="129d3-200">Futtassa az alábbi PowerShell-parancsot:</span><span class="sxs-lookup"><span data-stu-id="129d3-200">Run the following PowerShell command:</span></span>
   
    ```
    Get-AzureKeyVaultSecret <<key vault name>> Redis--Configuration | Select-Object *
    ```

3. <span data-ttu-id="129d3-201">Futtassa újra az adatbázis-kapcsolati karakterlánc hozzáadása a SetupKeyVault.ps:</span><span class="sxs-lookup"><span data-stu-id="129d3-201">Run SetupKeyVault.ps again to add the database connection string:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Data--SurveysConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```
   
    <span data-ttu-id="129d3-202">ahol `<<DB connection string>>` van az adatbázis-kapcsolati karakterlánc értékét.</span><span class="sxs-lookup"><span data-stu-id="129d3-202">where `<<DB connection string>>` is the value of the database connection string.</span></span>
   
    <span data-ttu-id="129d3-203">A helyi adatbázisban való teszteléshez, másolja a kapcsolati karakterláncot a Tailspin.Surveys.Web/appsettings.json fájlból.</span><span class="sxs-lookup"><span data-stu-id="129d3-203">For testing with the local database, copy the connection string from the Tailspin.Surveys.Web/appsettings.json file.</span></span> <span data-ttu-id="129d3-204">Ha így tesz, ügyeljen arra, hogy módosítsa a dupla fordított perjel ('\\\\"), egy fordított perjel.</span><span class="sxs-lookup"><span data-stu-id="129d3-204">If you do that, make sure to change the double backslash ('\\\\') into a single backslash.</span></span> <span data-ttu-id="129d3-205">A dupla fordított perjelet karakteréhez escape-karakter, a JSON-fájlban.</span><span class="sxs-lookup"><span data-stu-id="129d3-205">The double backslash is an escape character in the JSON file.</span></span>
   
    <span data-ttu-id="129d3-206">Példa:</span><span class="sxs-lookup"><span data-stu-id="129d3-206">Example:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName Data--SurveysConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true" 
    ```

### <a name="uncomment-the-code-that-enables-key-vault"></a><span data-ttu-id="129d3-207">Állítsa vissza a kódot, amely lehetővé teszi, hogy a Key Vault</span><span class="sxs-lookup"><span data-stu-id="129d3-207">Uncomment the code that enables Key Vault</span></span>
1. <span data-ttu-id="129d3-208">Nyissa meg a Tailspin.Surveys megoldást.</span><span class="sxs-lookup"><span data-stu-id="129d3-208">Open the Tailspin.Surveys solution.</span></span>
2. <span data-ttu-id="129d3-209">Keresse meg a következő kódrészletet Tailspin.Surveys.Web/Startup.cs, és állítsa vissza azt.</span><span class="sxs-lookup"><span data-stu-id="129d3-209">In Tailspin.Surveys.Web/Startup.cs, locate the following code block and uncomment it.</span></span>
   
    ```csharp
    //var config = builder.Build();
    //builder.AddAzureKeyVault(
    //    $"https://{config["KeyVault:Name"]}.vault.azure.net/",
    //    config["AzureAd:ClientId"],
    //    config["AzureAd:ClientSecret"]);
    ```
3. <span data-ttu-id="129d3-210">Tailspin.Surveys.Web/Startup.cs, keresse meg a kódot, amely regisztrálja a `ICredentialService`.</span><span class="sxs-lookup"><span data-stu-id="129d3-210">In Tailspin.Surveys.Web/Startup.cs, locate the code that registers the `ICredentialService`.</span></span> <span data-ttu-id="129d3-211">Állítsa vissza a sort, amely `CertificateCredentialService`, és tegye megjegyzésbe a sor használó `ClientCredentialService`:</span><span class="sxs-lookup"><span data-stu-id="129d3-211">Uncomment the line that uses `CertificateCredentialService`, and comment out the line that uses `ClientCredentialService`:</span></span>
   
    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```
   
    <span data-ttu-id="129d3-212">Ez a változás lehetővé teszi a WebApp [ügyféltény] [ client-assertion] az OAuth hozzáférési tokenek beszerzése.</span><span class="sxs-lookup"><span data-stu-id="129d3-212">This change enables the web app to use [Client assertion][client-assertion] to get OAuth access tokens.</span></span> <span data-ttu-id="129d3-213">Az ügyféltény az OAuth ügyfélkulcs nem szükséges.</span><span class="sxs-lookup"><span data-stu-id="129d3-213">With client assertion, you don't need an OAuth client secret.</span></span> <span data-ttu-id="129d3-214">Másik lehetőségként tárolhatja a titkos ügyfélkulcsot a key vaultban.</span><span class="sxs-lookup"><span data-stu-id="129d3-214">Alternatively, you could store the client secret in key vault.</span></span> <span data-ttu-id="129d3-215">Azonban a key vault és ügyféltény mindkettő az ügyfél használja a tanúsítvány, hogy legyen, ha engedélyezi a key vault, célszerű ügyféltény is engedélyezi.</span><span class="sxs-lookup"><span data-stu-id="129d3-215">However, key vault and client assertion both use a client certificate, so if you enable key vault, it's a good practice to enable client assertion as well.</span></span>

### <a name="update-the-user-secrets"></a><span data-ttu-id="129d3-216">A felhasználó titkos kódok frissítése</span><span class="sxs-lookup"><span data-stu-id="129d3-216">Update the user secrets</span></span>
<span data-ttu-id="129d3-217">A Megoldáskezelőben kattintson a jobb gombbal a Tailspin.Surveys.Web projektet, és válassza ki **felhasználói titkok kezelése**.</span><span class="sxs-lookup"><span data-stu-id="129d3-217">In Solution Explorer, right-click the Tailspin.Surveys.Web project and select **Manage User Secrets**.</span></span> <span data-ttu-id="129d3-218">A secrets.json fájlban törölje a meglévő JSON, és illessze be a következő:</span><span class="sxs-lookup"><span data-stu-id="129d3-218">In the secrets.json file, delete the existing JSON and paste in the following:</span></span>

```json
{
  "AzureAd": {
    "ClientId": "[Surveys web app client ID]",
    "ClientSecret": "[Surveys web app client secret]",
    "PostLogoutRedirectUri": "https://localhost:44300/",
    "WebApiResourceId": "[App ID URI of your Surveys.WebAPI application]",
    "Asymmetric": {
      "CertificateThumbprint": "[certificate thumbprint. Example: 105b2ff3bc842c53582661716db1b7cdc6b43ec9]",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "ValidationRequired": "false"
    }
  },
  "KeyVault": {
    "Name": "[key vault name]"
  }
}
```

<span data-ttu-id="129d3-219">Cserélje le a [szögletes zárójelek] bejegyzést a megfelelő értékekkel.</span><span class="sxs-lookup"><span data-stu-id="129d3-219">Replace the entries in [square brackets] with the correct values.</span></span>

* <span data-ttu-id="129d3-220">`AzureAd:ClientId`: Az a Surveys alkalmazás ügyfél-azonosító.</span><span class="sxs-lookup"><span data-stu-id="129d3-220">`AzureAd:ClientId`: The client ID of the Surveys app.</span></span>
* <span data-ttu-id="129d3-221">`AzureAd:ClientSecret`: A kulcs, amely akkor jön létre, ha a Surveys alkalmazás Azure AD-ben regisztrált.</span><span class="sxs-lookup"><span data-stu-id="129d3-221">`AzureAd:ClientSecret`: The key that you generated when you registered the Surveys application in Azure AD.</span></span>
* <span data-ttu-id="129d3-222">`AzureAd:WebApiResourceId`: Az Alkalmazásazonosító URI-t, hogy a megadott Azure AD-ben a Surveys.WebAPI alkalmazás létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="129d3-222">`AzureAd:WebApiResourceId`: The App ID URI that you specified when you created the Surveys.WebAPI application in Azure AD.</span></span>
* <span data-ttu-id="129d3-223">`Asymmetric:CertificateThumbprint`: A tanúsítvány ujjlenyomatát a korábban, az ügyféltanúsítvány létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="129d3-223">`Asymmetric:CertificateThumbprint`: The certificate thumbprint that you got previously, when you created the client certificate.</span></span>
* <span data-ttu-id="129d3-224">`KeyVault:Name`: A key vault a neve.</span><span class="sxs-lookup"><span data-stu-id="129d3-224">`KeyVault:Name`: The name of your key vault.</span></span>

> [!NOTE]
> <span data-ttu-id="129d3-225">`Asymmetric:ValidationRequired` hamis értéket, mert a korábban létrehozott tanúsítvány nem volt aláírva a legfelső szintű hitelesítésszolgáltató (CA).</span><span class="sxs-lookup"><span data-stu-id="129d3-225">`Asymmetric:ValidationRequired` is false because the certificate that you created previously was not signed by a root certificate authority (CA).</span></span> <span data-ttu-id="129d3-226">Éles környezetben állítsa be, és a legfelső szintű hitelesítésszolgáltató által aláírt tanúsítványt használjon `ValidationRequired` igaz értékre.</span><span class="sxs-lookup"><span data-stu-id="129d3-226">In production, use a certificate that is signed by a root CA and set `ValidationRequired` to true.</span></span>
> 
> 

<span data-ttu-id="129d3-227">Mentse a frissített secrets.json fájlt.</span><span class="sxs-lookup"><span data-stu-id="129d3-227">Save the updated secrets.json file.</span></span>

<span data-ttu-id="129d3-228">Ezután a Megoldáskezelőben kattintson a jobb gombbal a Tailspin.Surveys.WebApi projektet, és válassza ki **felhasználói titkok kezelése**.</span><span class="sxs-lookup"><span data-stu-id="129d3-228">Next, in Solution Explorer, right-click the Tailspin.Surveys.WebApi project and select **Manage User Secrets**.</span></span> <span data-ttu-id="129d3-229">Törölje a meglévő JSON, és illessze be a következőket:</span><span class="sxs-lookup"><span data-stu-id="129d3-229">Delete the existing JSON and paste in the following:</span></span>

```json
{
  "AzureAd": {
    "ClientId": "[Surveys.WebAPI client ID]",
    "WebApiResourceId": "https://tailspin5.onmicrosoft.com/surveys.webapi",
    "Asymmetric": {
      "CertificateThumbprint": "[certificate thumbprint]",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "ValidationRequired": "false"
    }
  },
  "KeyVault": {
    "Name": "[key vault name]"
  }
}
```

<span data-ttu-id="129d3-230">Cserélje le a [szögletes zárójelek] bejegyzést, és mentse a secrets.json fájlt.</span><span class="sxs-lookup"><span data-stu-id="129d3-230">Replace the entries in [square brackets] and save the secrets.json file.</span></span>

> [!NOTE]
> <span data-ttu-id="129d3-231">A webes API ügyeljen arra, hogy az ügyfél-Azonosítót használja a Surveys.WebAPI alkalmazás, nem a Surveys alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="129d3-231">For the web API, make sure to use the client ID for the Surveys.WebAPI application, not the Surveys application.</span></span>
> 
> 

<span data-ttu-id="129d3-232">[**Tovább**][adfs]</span><span class="sxs-lookup"><span data-stu-id="129d3-232">[**Next**][adfs]</span></span>

<!-- Links -->
[adfs]: ./adfs.md
[authorize-app]: /azure/key-vault/key-vault-get-started//#authorize
[azure-portal]: https://portal.azure.com
[azure-rm-cmdlets]: https://msdn.microsoft.com/library/mt125356.aspx
[client-assertion]: client-assertion.md
[configuration]: /aspnet/core/fundamentals/configuration
[KeyVault]: https://azure.microsoft.com/services/key-vault/
[key-tags]: https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_Keytags
[Microsoft.Azure.KeyVault]: https://www.nuget.org/packages/Microsoft.Azure.KeyVault/
[options]: /aspnet/core/fundamentals/configuration#using-options-and-configuration-objects
[readme]: ./run-the-app.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[user-secrets]: /aspnet/core/security/app-secrets
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
