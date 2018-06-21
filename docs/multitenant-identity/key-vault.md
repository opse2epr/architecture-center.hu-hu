---
title: Key Vault segítségével alkalmazás titkos kulcsok védelme
description: A Key Vault szolgáltatás használata az alkalmazás titkos kulcsok tárolására
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
ms.openlocfilehash: d49129a38d0413f6006095f03b817885e1ce6c92
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/16/2018
ms.locfileid: "31012518"
---
# <a name="use-azure-key-vault-to-protect-application-secrets"></a><span data-ttu-id="7714e-103">Az Azure Key Vault segítségével alkalmazás titkos kulcsok védelme</span><span class="sxs-lookup"><span data-stu-id="7714e-103">Use Azure Key Vault to protect application secrets</span></span>

<span data-ttu-id="7714e-104">[![GitHub](../_images/github.png) Mintakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="7714e-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="7714e-105">Esetében gyakori, hogy bizalmas és védeni kell, például alkalmazás beállításait:</span><span class="sxs-lookup"><span data-stu-id="7714e-105">It's common to have application settings that are sensitive and must be protected, such as:</span></span>

* <span data-ttu-id="7714e-106">Adatbázisok kapcsolati karakterláncai</span><span class="sxs-lookup"><span data-stu-id="7714e-106">Database connection strings</span></span>
* <span data-ttu-id="7714e-107">Jelszavak</span><span class="sxs-lookup"><span data-stu-id="7714e-107">Passwords</span></span>
* <span data-ttu-id="7714e-108">Titkosítási kulcsok</span><span class="sxs-lookup"><span data-stu-id="7714e-108">Cryptographic keys</span></span>

<span data-ttu-id="7714e-109">Biztonsági szempontból ajánlott a verziókövetési rendszerrel ezeknek a kulcsoknak soha nem tárolja.</span><span class="sxs-lookup"><span data-stu-id="7714e-109">As a security best practice, you should never store these secrets in source control.</span></span> <span data-ttu-id="7714e-110">Annak érdekében, hogy nyilvánosságra kerüljenek túl könnyű &mdash; akkor is, ha a forráskódraktárban személyes.</span><span class="sxs-lookup"><span data-stu-id="7714e-110">It's too easy for them to leak &mdash; even if your source code repository is private.</span></span> <span data-ttu-id="7714e-111">És azt nem szinte megakadályozza a titkos kulcsok a főkönyvi nyilvános.</span><span class="sxs-lookup"><span data-stu-id="7714e-111">And it's not just about keeping secrets from the general public.</span></span> <span data-ttu-id="7714e-112">A nagyobb projektek mely fejlesztők korlátozni lehet, hogy szeretné, és operátorok férhetnek hozzá a termelési titkos kulcsok.</span><span class="sxs-lookup"><span data-stu-id="7714e-112">On larger projects, you might want to restrict which developers and operators can access the production secrets.</span></span> <span data-ttu-id="7714e-113">(A teszt- vagy környezetekben található beállítások eltérnek.)</span><span class="sxs-lookup"><span data-stu-id="7714e-113">(Settings for test or development environments are different.)</span></span>

<span data-ttu-id="7714e-114">A titkos adatok tárolására egy biztonságosabb beállítás [Azure Key Vault][KeyVault].</span><span class="sxs-lookup"><span data-stu-id="7714e-114">A more secure option is to store these secrets in [Azure Key Vault][KeyVault].</span></span> <span data-ttu-id="7714e-115">Key Vault egy a felhőben üzemeltetett szolgáltatás kriptográfiai kulcsokat és más titkos kulcsok kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="7714e-115">Key Vault is a cloud-hosted service for managing cryptographic keys and other secrets.</span></span> <span data-ttu-id="7714e-116">Ez a cikk bemutatja, hogyan Key Vault használatával tárolja az alkalmazás konfigurációs beállításokat.</span><span class="sxs-lookup"><span data-stu-id="7714e-116">This article shows how to use Key Vault to store configuration settings for your app.</span></span>

<span data-ttu-id="7714e-117">Az a [Dejójáték felmérések] [ Surveys] alkalmazás, a következő beállítások érhetők titkos:</span><span class="sxs-lookup"><span data-stu-id="7714e-117">In the [Tailspin Surveys][Surveys] application, the following settings are secret:</span></span>

* <span data-ttu-id="7714e-118">Az adatbázis-kapcsolati karakterlánc.</span><span class="sxs-lookup"><span data-stu-id="7714e-118">The database connection string.</span></span>
* <span data-ttu-id="7714e-119">A Redis kapcsolódási karakterlánc.</span><span class="sxs-lookup"><span data-stu-id="7714e-119">The Redis connection string.</span></span>
* <span data-ttu-id="7714e-120">A webes alkalmazás titkos ügyfélkódot.</span><span class="sxs-lookup"><span data-stu-id="7714e-120">The client secret for the web application.</span></span>

<span data-ttu-id="7714e-121">A felmérések alkalmazás betölt a konfigurációs beállításokat a következő helyről:</span><span class="sxs-lookup"><span data-stu-id="7714e-121">The Surveys application loads configuration settings from the following places:</span></span>

* <span data-ttu-id="7714e-122">A appsettings.json fájl</span><span class="sxs-lookup"><span data-stu-id="7714e-122">The appsettings.json file</span></span>
* <span data-ttu-id="7714e-123">A [felhasználói titkos kulcsok tárolási] [ user-secrets] (fejlesztőkörnyezet; csak tesztelési célra)</span><span class="sxs-lookup"><span data-stu-id="7714e-123">The [user secrets store][user-secrets] (development environment only; for testing)</span></span>
* <span data-ttu-id="7714e-124">Az üzemeltetési környezet (az Azure web apps Alkalmazásbeállítások)</span><span class="sxs-lookup"><span data-stu-id="7714e-124">The hosting environment (app settings in Azure web apps)</span></span>
* <span data-ttu-id="7714e-125">Key Vault (Ha engedélyezve van)</span><span class="sxs-lookup"><span data-stu-id="7714e-125">Key Vault (when enabled)</span></span>

<span data-ttu-id="7714e-126">Minden egyes ezeket a felülbírálásokat az előzőre, így a Key Vault tárolt beállítások elsőbbséget.</span><span class="sxs-lookup"><span data-stu-id="7714e-126">Each of these overrides the previous one, so any settings stored in Key Vault take precedence.</span></span>

> [!NOTE]
> <span data-ttu-id="7714e-127">Alapértelmezés szerint le van tiltva a Key Vault konfigurációszolgáltatót.</span><span class="sxs-lookup"><span data-stu-id="7714e-127">By default, the Key Vault configuration provider is disabled.</span></span> <span data-ttu-id="7714e-128">Nincs szükség az alkalmazás helyi futtatásához.</span><span class="sxs-lookup"><span data-stu-id="7714e-128">It's not needed for running the application locally.</span></span> <span data-ttu-id="7714e-129">Éles környezetben azt szeretné engedélyezni.</span><span class="sxs-lookup"><span data-stu-id="7714e-129">You would enable it in a production deployment.</span></span>

<span data-ttu-id="7714e-130">Indításkor az alkalmazás beállítást olvassa ki minden regisztrált konfigurációszolgáltatót, és egy szigorú típusmegadású beállítások objektum feltöltéséhez használja őket.</span><span class="sxs-lookup"><span data-stu-id="7714e-130">At startup, the application reads settings from every registered configuration provider, and uses them to populate a strongly typed options object.</span></span> <span data-ttu-id="7714e-131">További információkért lásd: [lehetőségek használata és konfigurációs objektumok][options].</span><span class="sxs-lookup"><span data-stu-id="7714e-131">For more information, see [Using Options and configuration objects][options].</span></span>

## <a name="setting-up-key-vault-in-the-surveys-app"></a><span data-ttu-id="7714e-132">Key Vault a felmérések alkalmazás beállítása</span><span class="sxs-lookup"><span data-stu-id="7714e-132">Setting up Key Vault in the Surveys app</span></span>
<span data-ttu-id="7714e-133">Előfeltételek:</span><span class="sxs-lookup"><span data-stu-id="7714e-133">Prerequisites:</span></span>

* <span data-ttu-id="7714e-134">Telepítse a [Azure Resource Manager parancsmagjainak][azure-rm-cmdlets].</span><span class="sxs-lookup"><span data-stu-id="7714e-134">Install the [Azure Resource Manager Cmdlets][azure-rm-cmdlets].</span></span>
* <span data-ttu-id="7714e-135">Konfigurálja az felmérések alkalmazást, a [futtassa a felmérések alkalmazást][readme].</span><span class="sxs-lookup"><span data-stu-id="7714e-135">Configure the Surveys application as described in [Run the Surveys application][readme].</span></span>

<span data-ttu-id="7714e-136">Magas szintű lépéseket:</span><span class="sxs-lookup"><span data-stu-id="7714e-136">High-level steps:</span></span>

1. <span data-ttu-id="7714e-137">Állítson be egy rendszergazdai jogú felhasználót a bérlőben.</span><span class="sxs-lookup"><span data-stu-id="7714e-137">Set up an admin user in the tenant.</span></span>
2. <span data-ttu-id="7714e-138">Állítson be egy ügyféltanúsítványt.</span><span class="sxs-lookup"><span data-stu-id="7714e-138">Set up a client certificate.</span></span>
3. <span data-ttu-id="7714e-139">Hozzon létre egy kulcstartót.</span><span class="sxs-lookup"><span data-stu-id="7714e-139">Create a key vault.</span></span>
4. <span data-ttu-id="7714e-140">Konfigurációs beállítások hozzáadása a kulcstartót.</span><span class="sxs-lookup"><span data-stu-id="7714e-140">Add configuration settings to your key vault.</span></span>
5. <span data-ttu-id="7714e-141">Állítsa vissza a kódot, amely lehetővé teszi, hogy a kulcstároló.</span><span class="sxs-lookup"><span data-stu-id="7714e-141">Uncomment the code that enables key vault.</span></span>
6. <span data-ttu-id="7714e-142">Frissítse az alkalmazás felhasználói titkos kulcsok.</span><span class="sxs-lookup"><span data-stu-id="7714e-142">Update the application's user secrets.</span></span>

### <a name="set-up-an-admin-user"></a><span data-ttu-id="7714e-143">Állítson be egy rendszergazdai jogú felhasználó</span><span class="sxs-lookup"><span data-stu-id="7714e-143">Set up an admin user</span></span>
> [!NOTE]
> <span data-ttu-id="7714e-144">Hozzon létre egy kulcstartót, kezelheti az Azure-előfizetéshez fiókot kell használnia.</span><span class="sxs-lookup"><span data-stu-id="7714e-144">To create a key vault, you must use an account which can manage your Azure subscription.</span></span> <span data-ttu-id="7714e-145">Emellett bármely alkalmazás, amely a key vault olvasni engedélyeznie kell regisztrálni az ugyanannak a bérlőnek, mint ez a fiók.</span><span class="sxs-lookup"><span data-stu-id="7714e-145">Also, any application that you authorize to read from the key vault must be registered in the same tenant as that account.</span></span>
> 
> 

<span data-ttu-id="7714e-146">Ebben a lépésben teheti meg arról, hogy is létrehozott egy kulcstartót, miközben a bérlőtől felhasználóként van bejelentkezve az ahol az felmérések alkalmazás a regisztrációt.</span><span class="sxs-lookup"><span data-stu-id="7714e-146">In this step, you will make sure that you can create a key vault while signed in as a user from the tenant where the Surveys app is registered.</span></span>

<span data-ttu-id="7714e-147">Hozzon létre egy rendszergazda felhasználó, ahol a felmérések alkalmazás regisztrálva van az Azure AD-bérlő belül.</span><span class="sxs-lookup"><span data-stu-id="7714e-147">Create an administrator user within the Azure AD tenant where the Surveys application is registered.</span></span>

1. <span data-ttu-id="7714e-148">Jelentkezzen be a [Azure-portálon][azure-portal].</span><span class="sxs-lookup"><span data-stu-id="7714e-148">Log into the [Azure portal][azure-portal].</span></span>
2. <span data-ttu-id="7714e-149">Válassza ki az Azure AD-bérlő, ahol az alkalmazás regisztrálva van.</span><span class="sxs-lookup"><span data-stu-id="7714e-149">Select the Azure AD tenant where your application is registered.</span></span>
3. <span data-ttu-id="7714e-150">Kattintson a **további szolgáltatás** > **biztonság + identitás szakaszában** > **Azure Active Directory** > **felhasználók és csoportok**   >  **Minden felhasználó**.</span><span class="sxs-lookup"><span data-stu-id="7714e-150">Click **More service** > **SECURITY + IDENTITY** > **Azure Active Directory** > **User and groups** > **All users**.</span></span>
4. <span data-ttu-id="7714e-151">A portál felső részén kattintson **új felhasználó**.</span><span class="sxs-lookup"><span data-stu-id="7714e-151">At the top of the portal, click **New user**.</span></span>
5. <span data-ttu-id="7714e-152">Töltse ki a mezőket, és rendelje hozzá a felhasználót a **globális rendszergazda** címtár szerepkörrel.</span><span class="sxs-lookup"><span data-stu-id="7714e-152">Fill in the fields and assign the user to the **Global administrator** directory role.</span></span>
6. <span data-ttu-id="7714e-153">Kattintson a **Create** (Létrehozás) gombra.</span><span class="sxs-lookup"><span data-stu-id="7714e-153">Click **Create**.</span></span>

![Globális rendszergazdai jogú felhasználó](./images/running-the-app/global-admin-user.png)

<span data-ttu-id="7714e-155">Most már az előfizetés tulajdonosa a felhasználóhoz rendelni.</span><span class="sxs-lookup"><span data-stu-id="7714e-155">Now assign this user as the subscription owner.</span></span>

1. <span data-ttu-id="7714e-156">A központ menüben válassza ki a **előfizetések**.</span><span class="sxs-lookup"><span data-stu-id="7714e-156">On the Hub menu, select **Subscriptions**.</span></span>

    ![](./images/running-the-app/subscriptions.png)

2. <span data-ttu-id="7714e-157">Válassza ki az előfizetést, amelyet a rendszergazda eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="7714e-157">Select the subscription that you want the administrator to access.</span></span>
3. <span data-ttu-id="7714e-158">Az előfizetés panelen válassza ki a **hozzáférés-vezérlés (IAM)**.</span><span class="sxs-lookup"><span data-stu-id="7714e-158">In the subscription blade, select **Access control (IAM)**.</span></span>
4. <span data-ttu-id="7714e-159">Kattintson a **Hozzáadás** parancsra.</span><span class="sxs-lookup"><span data-stu-id="7714e-159">Click **Add**.</span></span>
4. <span data-ttu-id="7714e-160">A **szerepkör**, jelölje be **tulajdonos**.</span><span class="sxs-lookup"><span data-stu-id="7714e-160">Under **Role**, select **Owner**.</span></span>
5. <span data-ttu-id="7714e-161">Írja be a felhasználó hozzá szeretne adni a tulajdonos e-mail címe.</span><span class="sxs-lookup"><span data-stu-id="7714e-161">Type the email address of the user you want to add as owner.</span></span>
6. <span data-ttu-id="7714e-162">Válassza ki a felhasználót, és kattintson a **mentése**.</span><span class="sxs-lookup"><span data-stu-id="7714e-162">Select the user and click **Save**.</span></span>

### <a name="set-up-a-client-certificate"></a><span data-ttu-id="7714e-163">Ügyfél-tanúsítvány beállítása</span><span class="sxs-lookup"><span data-stu-id="7714e-163">Set up a client certificate</span></span>
1. <span data-ttu-id="7714e-164">A PowerShell-parancsprogrammal [/Scripts/Setup-KeyVault.ps1] [ Setup-KeyVault] az alábbiak szerint:</span><span class="sxs-lookup"><span data-stu-id="7714e-164">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    <span data-ttu-id="7714e-165">Az a `Subject` paraméter, adja meg a nevét, például a "surveysapp".</span><span class="sxs-lookup"><span data-stu-id="7714e-165">For the `Subject` parameter, enter any name, such as "surveysapp".</span></span> <span data-ttu-id="7714e-166">A parancsfájl létrehoz egy önaláírt tanúsítványt, és tárolja a "jelenlegi felhasználó/Personal" tanúsítványtárolójában.</span><span class="sxs-lookup"><span data-stu-id="7714e-166">The script generates a self-signed certificate and stores it in the "Current User/Personal" certificate store.</span></span> <span data-ttu-id="7714e-167">Parancsfájl eredménye a JSON-töredéket.</span><span class="sxs-lookup"><span data-stu-id="7714e-167">The output from the script is a JSON fragment.</span></span> <span data-ttu-id="7714e-168">Másolja ezt az értéket.</span><span class="sxs-lookup"><span data-stu-id="7714e-168">Copy this value.</span></span>

2. <span data-ttu-id="7714e-169">Az a [Azure-portálon][azure-portal], lépjen a mappába, ahol a a felmérések alkalmazás regisztrálva, a fiók kiválasztja a portál jobb felső sarkában.</span><span class="sxs-lookup"><span data-stu-id="7714e-169">In the [Azure portal][azure-portal], switch to the directory where the Surveys application is registered, by selecting your account in the top right corner of the portal.</span></span>

3. <span data-ttu-id="7714e-170">Válassza ki **az Azure Active Directory** > **App regisztrációk** > felmérések</span><span class="sxs-lookup"><span data-stu-id="7714e-170">Select **Azure Active Directory** > **App Registrations** > Surveys</span></span>

4.  <span data-ttu-id="7714e-171">Kattintson a **Manifest** , majd **szerkesztése**.</span><span class="sxs-lookup"><span data-stu-id="7714e-171">Click **Manifest** and then **Edit**.</span></span>

5.  <span data-ttu-id="7714e-172">Illessze be a parancsfájlt a kimenetét a `keyCredentials` tulajdonság.</span><span class="sxs-lookup"><span data-stu-id="7714e-172">Paste the output from the script into the `keyCredentials` property.</span></span> <span data-ttu-id="7714e-173">Ennek a következőképpen kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="7714e-173">It should look similar to the following:</span></span>
        
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

6. <span data-ttu-id="7714e-174">Kattintson a **Save** (Mentés) gombra.</span><span class="sxs-lookup"><span data-stu-id="7714e-174">Click **Save**.</span></span>  

7. <span data-ttu-id="7714e-175">Ismételje meg a 3-6 ugyanazon JSON töredék hozzáadása a Web API (Surveys.WebAPI) az alkalmazás jegyzékében.</span><span class="sxs-lookup"><span data-stu-id="7714e-175">Repeat steps 3-6 to add the same JSON fragment to the application manifest of the web API (Surveys.WebAPI).</span></span>

8. <span data-ttu-id="7714e-176">A PowerShell-ablakban futtassa a következő parancsot a tanúsítvány ujjlenyomatát.</span><span class="sxs-lookup"><span data-stu-id="7714e-176">From the PowerShell window, run the following command to get the thumbprint of the certificate.</span></span>
   
    ```
    certutil -store -user my [subject]
    ```
    
    <span data-ttu-id="7714e-177">A `[subject]`, a tulajdonos a PowerShell-parancsfájl a megadott értéket használja.</span><span class="sxs-lookup"><span data-stu-id="7714e-177">For `[subject]`, use the value that you specified for Subject in the PowerShell script.</span></span> <span data-ttu-id="7714e-178">Az ujjlenyomat megtalálható-e a "Cert Hash(sha1)".</span><span class="sxs-lookup"><span data-stu-id="7714e-178">The thumbprint is listed under "Cert Hash(sha1)".</span></span> <span data-ttu-id="7714e-179">Másolja ezt az értéket.</span><span class="sxs-lookup"><span data-stu-id="7714e-179">Copy this value.</span></span> <span data-ttu-id="7714e-180">Az ujjlenyomat később fogja használni.</span><span class="sxs-lookup"><span data-stu-id="7714e-180">You will use the thumbprint later.</span></span>

### <a name="create-a-key-vault"></a><span data-ttu-id="7714e-181">Kulcstartó létrehozása</span><span class="sxs-lookup"><span data-stu-id="7714e-181">Create a key vault</span></span>
1. <span data-ttu-id="7714e-182">A PowerShell-parancsprogrammal [/Scripts/Setup-KeyVault.ps1] [ Setup-KeyVault] az alábbiak szerint:</span><span class="sxs-lookup"><span data-stu-id="7714e-182">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```
   
    <span data-ttu-id="7714e-183">Ha a hitelesítő adatok megadását kéri, jelentkezzen be az Azure AD-felhasználó, korábban létrehozott.</span><span class="sxs-lookup"><span data-stu-id="7714e-183">When prompted for credentials, sign in as the Azure AD user that you created earlier.</span></span> <span data-ttu-id="7714e-184">A parancsfájl létrehoz egy új erőforráscsoportot, és egy új kulcstartó adott erőforráscsoporton belül.</span><span class="sxs-lookup"><span data-stu-id="7714e-184">The script creates a new resource group, and a new key vault within that resource group.</span></span> 
   
2. <span data-ttu-id="7714e-185">Futtassa újra SetupKeyVault.ps az alábbiak szerint:</span><span class="sxs-lookup"><span data-stu-id="7714e-185">Run SetupKeyVault.ps again as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<Surveys app id>>", "<<Surveys.WebAPI app ID>>")
    ```
   
    <span data-ttu-id="7714e-186">Állítsa be a következő paraméterértékeket:</span><span class="sxs-lookup"><span data-stu-id="7714e-186">Set the following parameter values:</span></span>
   
       * <span data-ttu-id="7714e-187">kulcstároló neve = a neve, mint a kulcstároló az előző lépésben.</span><span class="sxs-lookup"><span data-stu-id="7714e-187">key vault name = The name that you gave the key vault in the previous step.</span></span>
       * <span data-ttu-id="7714e-188">Alkalmazásazonosító felmérések = az Alkalmazásazonosítót a felmérések webalkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="7714e-188">Surveys app ID = The application ID for the Surveys web application.</span></span>
       * <span data-ttu-id="7714e-189">Surveys.WebApi app ID = az Alkalmazásazonosítót a Surveys.WebAPI alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="7714e-189">Surveys.WebApi app ID = The application ID for the Surveys.WebAPI application.</span></span>
         
    <span data-ttu-id="7714e-190">Példa:</span><span class="sxs-lookup"><span data-stu-id="7714e-190">Example:</span></span>
     
    ```
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
    ```
    
    <span data-ttu-id="7714e-191">Ezt a parancsfájlt a webes alkalmazás és a titkos kulcsok lekérése a kulcstartót webes API engedélyezi.</span><span class="sxs-lookup"><span data-stu-id="7714e-191">This script authorizes the web app and web API to retrieve secrets from your key vault.</span></span> <span data-ttu-id="7714e-192">Lásd: [Ismerkedés az Azure Key Vault](/azure/key-vault/key-vault-get-started/) további információt.</span><span class="sxs-lookup"><span data-stu-id="7714e-192">See [Get started with Azure Key Vault](/azure/key-vault/key-vault-get-started/) for more information.</span></span>

### <a name="add-configuration-settings-to-your-key-vault"></a><span data-ttu-id="7714e-193">Konfigurációs beállítások hozzáadása a kulcstartót</span><span class="sxs-lookup"><span data-stu-id="7714e-193">Add configuration settings to your key vault</span></span>
1. <span data-ttu-id="7714e-194">Futtassa a következőképpen SetupKeyVault.ps::</span><span class="sxs-lookup"><span data-stu-id="7714e-194">Run SetupKeyVault.ps as follows::</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Redis--Configuration -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true" 
    ```
    <span data-ttu-id="7714e-195">Ha</span><span class="sxs-lookup"><span data-stu-id="7714e-195">where</span></span>
   
   * <span data-ttu-id="7714e-196">kulcstároló neve = a neve, mint a kulcstároló az előző lépésben.</span><span class="sxs-lookup"><span data-stu-id="7714e-196">key vault name = The name that you gave the key vault in the previous step.</span></span>
   * <span data-ttu-id="7714e-197">DNS-név redis = a Redis cache példány DNS-neve.</span><span class="sxs-lookup"><span data-stu-id="7714e-197">Redis DNS name = The DNS name of your Redis cache instance.</span></span>
   * <span data-ttu-id="7714e-198">Redis hozzáférési kulcsot a hozzáférési kulcsot a Redis cache példány =.</span><span class="sxs-lookup"><span data-stu-id="7714e-198">Redis access key = The access key for your Redis cache instance.</span></span>
     
2. <span data-ttu-id="7714e-199">Ezen a ponton akkor érdemes annak megállapítására, hogy sikeresen tárolta a titkos kulcsok számára kulcstároló.</span><span class="sxs-lookup"><span data-stu-id="7714e-199">At this point, it's a good idea to test whether you successfully stored the secrets to key vault.</span></span> <span data-ttu-id="7714e-200">Futtassa az alábbi PowerShell-parancsot:</span><span class="sxs-lookup"><span data-stu-id="7714e-200">Run the following PowerShell command:</span></span>
   
    ```
    Get-AzureKeyVaultSecret <<key vault name>> Redis--Configuration | Select-Object *
    ```

3. <span data-ttu-id="7714e-201">Futtassa újra az adatbázis-kapcsolati karakterlánc hozzáadása a SetupKeyVault.ps:</span><span class="sxs-lookup"><span data-stu-id="7714e-201">Run SetupKeyVault.ps again to add the database connection string:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Data--SurveysConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```
   
    <span data-ttu-id="7714e-202">Ha `<<DB connection string>>` van az adatbázis-kapcsolati karakterlánc értékét.</span><span class="sxs-lookup"><span data-stu-id="7714e-202">where `<<DB connection string>>` is the value of the database connection string.</span></span>
   
    <span data-ttu-id="7714e-203">A helyi adatbázis tesztelték, másolja a kapcsolati karakterláncot a Tailspin.Surveys.Web/appsettings.json fájlból.</span><span class="sxs-lookup"><span data-stu-id="7714e-203">For testing with the local database, copy the connection string from the Tailspin.Surveys.Web/appsettings.json file.</span></span> <span data-ttu-id="7714e-204">Ha így tesz, ügyeljen arra, hogy módosítsa a dupla fordított perjel ('\\\\") az egyetlen perjellel.</span><span class="sxs-lookup"><span data-stu-id="7714e-204">If you do that, make sure to change the double backslash ('\\\\') into a single backslash.</span></span> <span data-ttu-id="7714e-205">A kettős fordított karakteréhez escape-karakter, a JSON-fájlban.</span><span class="sxs-lookup"><span data-stu-id="7714e-205">The double backslash is an escape character in the JSON file.</span></span>
   
    <span data-ttu-id="7714e-206">Példa:</span><span class="sxs-lookup"><span data-stu-id="7714e-206">Example:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName Data--SurveysConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true" 
    ```

### <a name="uncomment-the-code-that-enables-key-vault"></a><span data-ttu-id="7714e-207">Állítsa vissza a kódot, amely lehetővé teszi, hogy a Key Vault</span><span class="sxs-lookup"><span data-stu-id="7714e-207">Uncomment the code that enables Key Vault</span></span>
1. <span data-ttu-id="7714e-208">Nyissa meg a Tailspin.Surveys megoldást.</span><span class="sxs-lookup"><span data-stu-id="7714e-208">Open the Tailspin.Surveys solution.</span></span>
2. <span data-ttu-id="7714e-209">A Tailspin.Surveys.Web/Startup.cs keresse meg a következő kódrészletet, és állítsa vissza azt.</span><span class="sxs-lookup"><span data-stu-id="7714e-209">In Tailspin.Surveys.Web/Startup.cs, locate the following code block and uncomment it.</span></span>
   
    ```csharp
    //var config = builder.Build();
    //builder.AddAzureKeyVault(
    //    $"https://{config["KeyVault:Name"]}.vault.azure.net/",
    //    config["AzureAd:ClientId"],
    //    config["AzureAd:ClientSecret"]);
    ```
3. <span data-ttu-id="7714e-210">Tailspin.Surveys.Web/Startup.cs, keresse meg a kódot, amely regisztrálja a `ICredentialService`.</span><span class="sxs-lookup"><span data-stu-id="7714e-210">In Tailspin.Surveys.Web/Startup.cs, locate the code that registers the `ICredentialService`.</span></span> <span data-ttu-id="7714e-211">Állítsa vissza a sor által használt `CertificateCredentialService`, és a sor által használt megjegyzésbe `ClientCredentialService`:</span><span class="sxs-lookup"><span data-stu-id="7714e-211">Uncomment the line that uses `CertificateCredentialService`, and comment out the line that uses `ClientCredentialService`:</span></span>
   
    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```
   
    <span data-ttu-id="7714e-212">A módosítás lehetővé teszi a webalkalmazás használandó [ügyfél helyességi feltétel] [ client-assertion] lekérni az OAuth jogkivonatot.</span><span class="sxs-lookup"><span data-stu-id="7714e-212">This change enables the web app to use [Client assertion][client-assertion] to get OAuth access tokens.</span></span> <span data-ttu-id="7714e-213">Az ügyfél helyességi feltétel az OAuth ügyfélkulcs nem szükséges.</span><span class="sxs-lookup"><span data-stu-id="7714e-213">With client assertion, you don't need an OAuth client secret.</span></span> <span data-ttu-id="7714e-214">Azt is megteheti a titkos ügyfélkulcs tárolhatja a key vaultban.</span><span class="sxs-lookup"><span data-stu-id="7714e-214">Alternatively, you could store the client secret in key vault.</span></span> <span data-ttu-id="7714e-215">Azonban kulcstartó és mindkettő használja, hogy egy ügyfél ügyfél helyességi feltétel tanúsítvány, ha engedélyezi a kulcstároló, hogy célszerű engedélyezni, valamint az ügyfél állítás.</span><span class="sxs-lookup"><span data-stu-id="7714e-215">However, key vault and client assertion both use a client certificate, so if you enable key vault, it's a good practice to enable client assertion as well.</span></span>

### <a name="update-the-user-secrets"></a><span data-ttu-id="7714e-216">A felhasználó titkos kulcsok frissítése</span><span class="sxs-lookup"><span data-stu-id="7714e-216">Update the user secrets</span></span>
<span data-ttu-id="7714e-217">A Megoldáskezelőben kattintson a jobb gombbal a Tailspin.Surveys.Web projektet, és válassza ki **felhasználói kulcsok kezelése**.</span><span class="sxs-lookup"><span data-stu-id="7714e-217">In Solution Explorer, right-click the Tailspin.Surveys.Web project and select **Manage User Secrets**.</span></span> <span data-ttu-id="7714e-218">A secrets.json fájlban törölje a meglévő JSON, és illessze be a következőket:</span><span class="sxs-lookup"><span data-stu-id="7714e-218">In the secrets.json file, delete the existing JSON and paste in the following:</span></span>

    ```
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

<span data-ttu-id="7714e-219">A bejegyzések [szögletes zárójelbe] cserélje le a megfelelő értékeket.</span><span class="sxs-lookup"><span data-stu-id="7714e-219">Replace the entries in [square brackets] with the correct values.</span></span>

* <span data-ttu-id="7714e-220">`AzureAd:ClientId`: Az a felmérések alkalmazás ügyfél-azonosító.</span><span class="sxs-lookup"><span data-stu-id="7714e-220">`AzureAd:ClientId`: The client ID of the Surveys app.</span></span>
* <span data-ttu-id="7714e-221">`AzureAd:ClientSecret`: A kulcs az Azure ad-ben a felmérések alkalmazás regisztrálásakor létrehozott.</span><span class="sxs-lookup"><span data-stu-id="7714e-221">`AzureAd:ClientSecret`: The key that you generated when you registered the Surveys application in Azure AD.</span></span>
* <span data-ttu-id="7714e-222">`AzureAd:WebApiResourceId`: A App ID URI Azure AD-ben a Surveys.WebAPI alkalmazás létrehozásakor megadott.</span><span class="sxs-lookup"><span data-stu-id="7714e-222">`AzureAd:WebApiResourceId`: The App ID URI that you specified when you created the Surveys.WebAPI application in Azure AD.</span></span>
* <span data-ttu-id="7714e-223">`Asymmetric:CertificateThumbprint`: A tanúsítvány ujjlenyomata portáltól korábban, az ügyfél-tanúsítvány létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="7714e-223">`Asymmetric:CertificateThumbprint`: The certificate thumbprint that you got previously, when you created the client certificate.</span></span>
* <span data-ttu-id="7714e-224">`KeyVault:Name`: A kulcstartót a neve.</span><span class="sxs-lookup"><span data-stu-id="7714e-224">`KeyVault:Name`: The name of your key vault.</span></span>

> [!NOTE]
> <span data-ttu-id="7714e-225">`Asymmetric:ValidationRequired` hamis, mert a korábban létrehozott tanúsítvány nem volt által aláírt egy legfelső szintű hitelesítésszolgáltatót (CA).</span><span class="sxs-lookup"><span data-stu-id="7714e-225">`Asymmetric:ValidationRequired` is false because the certificate that you created previously was not signed by a root certificate authority (CA).</span></span> <span data-ttu-id="7714e-226">Éles, állítsa be, és a legfelső szintű hitelesítésszolgáltató által aláírt tanúsítványt használjon `ValidationRequired` igaz értékre.</span><span class="sxs-lookup"><span data-stu-id="7714e-226">In production, use a certificate that is signed by a root CA and set `ValidationRequired` to true.</span></span>
> 
> 

<span data-ttu-id="7714e-227">Mentse a frissített secrets.json fájlt.</span><span class="sxs-lookup"><span data-stu-id="7714e-227">Save the updated secrets.json file.</span></span>

<span data-ttu-id="7714e-228">Ezután a Megoldáskezelőben kattintson a jobb gombbal a Tailspin.Surveys.WebApi projektet, és válassza ki **felhasználói kulcsok kezelése**.</span><span class="sxs-lookup"><span data-stu-id="7714e-228">Next, in Solution Explorer, right-click the Tailspin.Surveys.WebApi project and select **Manage User Secrets**.</span></span> <span data-ttu-id="7714e-229">Törölje a meglévő JSON, és illessze be a következő:</span><span class="sxs-lookup"><span data-stu-id="7714e-229">Delete the existing JSON and paste in the following:</span></span>

```
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

<span data-ttu-id="7714e-230">Cserélje le a [szögletes zárójelbe] bejegyzések, és mentse a secrets.json fájlt.</span><span class="sxs-lookup"><span data-stu-id="7714e-230">Replace the entries in [square brackets] and save the secrets.json file.</span></span>

> [!NOTE]
> <span data-ttu-id="7714e-231">A webes API-t győződjön meg arról, az ügyfél-azonosító használata a Surveys.WebAPI alkalmazáshoz, nem a felmérések alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="7714e-231">For the web API, make sure to use the client ID for the Surveys.WebAPI application, not the Surveys application.</span></span>
> 
> 

<span data-ttu-id="7714e-232">[**Tovább**][adfs]</span><span class="sxs-lookup"><span data-stu-id="7714e-232">[**Next**][adfs]</span></span>

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
[user-secrets]: http://go.microsoft.com/fwlink/?LinkID=532709
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
