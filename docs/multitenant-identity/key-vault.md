---
title: Key Vault használata az alkalmazás titkainak védelmére
description: Tudnivalók a Key Vault szolgáltatás használatához az alkalmazás titkainak tárolásához.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
ms.openlocfilehash: 6aa8d33da0b2fd41fdc037bac28bca9f7ff09907
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299633"
---
# <a name="use-azure-key-vault-to-protect-application-secrets"></a>Azure Key Vault használata az alkalmazás titkainak védelmére

[![GitHub](../_images/github.png) Mintakód][sample application]

Célszerű a gyakori, hogy olyan alkalmazásbeállításokat, amelyek kis-és nagybetűket, és védeni kell, például:

* Adatbázisok kapcsolati sztringjei
* Jelszavak
* Titkosítási kulcsok

Bevált biztonsági gyakorlat verziókövetés titkos adatokat soha nem tárolja. Túl egyszerű, hogy nyilvánosságra kerüljenek &mdash; akkor is, ha a forráskód adattára privát. És azt nem szinte megakadályozza a titkos kulcsok az általános nyilvános. A nagyobb projektek mely fejlesztők korlátozni lehet, hogy szeretné, és operátorok hozzáférnek a termelési titkos kulcsait. (A tesztelési vagy fejlesztési környezetek beállítások eltérőek.)

Egy biztonságosabb megoldás, ha a titkos adatokat tárolni [Azure Key Vault][KeyVault]. A Key Vault egy olyan felhőben üzemeltetett szolgáltatás titkosítási kulcsok és egyéb titkok kezeléséhez. Ez a cikk bemutatja, hogyan tárolja a konfigurációs beállításokat az alkalmazás a Key Vault használatával.

Az a [Tailspin Surveys] [ Surveys] alkalmazás, a következő beállításokat is titkos:

* Az adatbázis-kapcsolati karakterláncot.
* A Redis kapcsolati karakterlánc.
* A titkos ügyfélkulcsot a webes alkalmazás.

A Surveys alkalmazás konfigurációs beállítások tölt be a következő helyeken:

* Az appsettings.json fájl
* A [felhasználói titkos kódok store] [ user-secrets] (fejlesztési környezet; csak tesztelési célra)
* Az üzemeltetési környezet (alkalmazás beállításai az Azure web apps)
* A Key Vault (Ha engedélyezve van)

Minden egyes ezeket a felülbírálásokat az előzőt, így a Key vaultban tárolt beállítások elsőbbséget.

> [!NOTE]
> A Key Vault konfigurációszolgáltató alapértelmezés szerint le van tiltva. Nincs szükség az alkalmazás helyi futtatásához. Éles környezetben azt szeretné engedélyezni.

Indításkor az alkalmazás minden regisztrált konfigurációszolgáltató beállításokat olvas, és feltölti egy szigorú típusmegadású objektum használja őket. További információkért lásd: [beállítások használatával és konfigurációs objektumok][options].

## <a name="setting-up-key-vault-in-the-surveys-app"></a>A Surveys alkalmazás Key Vault beállítása

Előfeltételek:

* Telepítse a [Azure Resource Manager parancsmagjainak][azure-rm-cmdlets].
* Konfigurálja a Surveys alkalmazás leírtak szerint [a Surveys alkalmazás futtatása][readme].

Magas szintű lépéseket:

1. Állítsa be egy rendszergazdai felhasználót a bérlőben.
2. Állítsa be az ügyféltanúsítványt.
3. Kulcstartó létrehozása.
4. Konfigurációs beállítások hozzáadása a kulcstartóhoz.
5. Állítsa vissza a kódot, amely lehetővé teszi, hogy a key vaultban.
6. Az alkalmazás felhasználói titkos kódok frissítése.

### <a name="set-up-an-admin-user"></a>Egy rendszergazda felhasználó beállítása

> [!NOTE]
> Hozzon létre egy kulcstartót, olyan fiók, amely az Azure-előfizetés kezelése céljából kell használnia. Is minden olyan alkalmazás, hogy olvassa el a key vaultban tárolt elérjék regisztrálni kell, hogy a fiók ugyanabban a bérlőben.

Ebben a lépésben teheti meg arról, hogy hozhat létre egy kulcstartót, amíg a bérlő felhasználóként jelentkezett be, a Surveys alkalmazás regisztrálva van-e.

Hozzon létre egy rendszergazda felhasználó belül az Azure AD-bérlővel, ahol a Surveys alkalmazás regisztrálva van-e.

1. Jelentkezzen be a [az Azure portal][azure-portal].
2. Válassza ki az Azure AD-bérlővel, ahol az alkalmazás regisztrálva lesz.
3. Kattintson a **további szolgáltatás** > **biztonság + IDENTITÁS** > **Azure Active Directory** > **felhasználók és csoportok**   >  **Minden felhasználó**.
4. Kattintson a portál tetején **új felhasználó**.
5. Töltse ki a mezőket, és rendelje hozzá a felhasználót a **globális rendszergazdai** címtárbeli szerepkör.
6. Kattintson a **Create** (Létrehozás) gombra.

![Globális rendszergazda](./images/running-the-app/global-admin-user.png)

Most rendelje hozzá a felhasználó az előfizetés tulajdonosaként.

1. A központ menüben válassza ki a **előfizetések**.

    ![Az Azure portal központ képernyőképe](./images/running-the-app/subscriptions.png)

2. A rendszergazda eléréséhez használni kívánt előfizetés kiválasztásához.
3. Az előfizetési panelen válassza ki a **hozzáférés-vezérlés (IAM)**.
4. Kattintson a **Hozzáadás** parancsra.
5. A **szerepkör**válassza **tulajdonosa**.
6. Írja be a tulajdonosként hozzáadni kívánt felhasználó e-mail-címét.
7. Válassza ki a felhasználót, és kattintson a **mentése**.

### <a name="set-up-a-client-certificate"></a>Ügyfél-tanúsítvány beállítása

1. A PowerShell-parancsprogrammal [/Scripts/Setup-KeyVault.ps1] [ Setup-KeyVault] módon:

    ```powershell
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    Az a `Subject` paramétert, tetszőleges nevet, például a "surveysapp" adja meg. A szkript létrehoz egy önaláírt tanúsítványt, és tárolja azokat az "aktuális User/Personal" tanúsítványtárolójában. A parancsprogram kimenete egy JSON-töredék. Másolja ezt az értéket.

2. Az a [az Azure portal][azure-portal], lépjen abba a könyvtárba, ahol a Surveys alkalmazás regisztrálva van, a portál jobb felső sarkában a fiók kiválasztásával.

3. Válassza ki **az Azure Active Directory** > **Alkalmazásregisztrációk** > felmérések

4. Kattintson a **Manifest** , majd **szerkesztése**.

5. Illessze be a szkript kimenetében a `keyCredentials` tulajdonság. Ennek a következőképpen kell kinéznie:

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

6. Kattintson a **Save** (Mentés) gombra.

7. Ismételje meg a 3-6 JSON ugyanarra a töredékre hozzáadása az alkalmazásjegyzékben, a webes API (Surveys.WebAPI).

8. A PowerShell-ablakban futtassa a következő parancsot a tanúsítvány ujjlenyomatának beolvasása.

    ```powershell
    certutil -store -user my [subject]
    ```

    A `[subject]`, használja az értéket, a PowerShell-parancsfájl a megadott tulajdonos. Az ujjlenyomat "Tanúsítvány SHA1" területen látható. Másolja ezt az értéket. Az ujjlenyomat később fogja használni.

### <a name="create-a-key-vault"></a>Kulcstartó létrehozása

1. A PowerShell-parancsprogrammal [/Scripts/Setup-KeyVault.ps1] [ Setup-KeyVault] módon:

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```

    Amikor a rendszer kéri a hitelesítő adatokat, jelentkezzen be az Azure AD-felhasználóként előzőleg létrehozott jelszóra. A szkript létrehoz egy új erőforráscsoportot, és a egy új kulcstartóba adott erőforráscsoporton belül.

2. A telepítő-KeyVault.ps1 ismét futtassa az alábbiak szerint:

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<Surveys app id>>", "<<Surveys.WebAPI app ID>>")
    ```

    Állítsa be a következő paraméter értékét:

       * a Key vault name = a neve, mint a key vault az előző lépésben.
       * Alkalmazásazonosító felmérések = az Alkalmazásazonosítót a felmérések webes alkalmazás.
       * Surveys.WebApi app ID = az Alkalmazásazonosítót a Surveys.WebAPI alkalmazáshoz.

    Példa:

    ```powershell
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
    ```

    Ez a szkript engedélyezi a webalkalmazás és webes API titkos kódok lekérése a key vaultban. Lásd: [első lépései az Azure Key Vault](/azure/key-vault/key-vault-get-started/) további információt.

### <a name="add-configuration-settings-to-your-key-vault"></a>Konfigurációs beállítások hozzáadása a kulcstartóhoz

1. Futtassa a telepítő-KeyVault.ps1 az alábbiak szerint:

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Redis--Configuration -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true"
    ```
    ahol

   * a Key vault name = a neve, mint a key vault az előző lépésben.
   * Redis DNS name = The DNS name of your Redis cache instance.
   * Redis Cache hozzáférési kulcs = a Redis cache-példány hozzáférési kulcsára.

2. Ezen a ponton célszerű annak megállapítására, hogy sikerült menteni a titkos kulcsok a key vaulttal. Futtassa az alábbi PowerShell-parancsot:

    ```powershell
    Get-AzureKeyVaultSecret <<key vault name>> Redis--Configuration | Select-Object *
    ```

3. Futtassa a telepítő-KeyVault.ps1 újra, hogy az adatbázis-kapcsolati sztring hozzáadása:

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Data--SurveysConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```

    ahol `<<DB connection string>>` van az adatbázis-kapcsolati karakterlánc értékét.

    A helyi adatbázisban való teszteléshez, másolja a kapcsolati karakterláncot a Tailspin.Surveys.Web/appsettings.json fájlból. Ha így tesz, ügyeljen arra, hogy módosítsa a dupla fordított perjel ('\\\\"), egy fordított perjel. A dupla fordított perjelet karakteréhez escape-karakter, a JSON-fájlban.

    Példa:

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName Data--SurveysConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true"
    ```

### <a name="uncomment-the-code-that-enables-key-vault"></a>Állítsa vissza a kódot, amely lehetővé teszi, hogy a Key Vault

1. Nyissa meg a Tailspin.Surveys megoldást.
2. Keresse meg a következő kódrészletet Tailspin.Surveys.Web/Startup.cs, és állítsa vissza azt.

    ```csharp
    //var config = builder.Build();
    //builder.AddAzureKeyVault(
    //    $"https://{config["KeyVault:Name"]}.vault.azure.net/",
    //    config["AzureAd:ClientId"],
    //    config["AzureAd:ClientSecret"]);
    ```
3. Tailspin.Surveys.Web/Startup.cs, keresse meg a kódot, amely regisztrálja a `ICredentialService`. Állítsa vissza a sort, amely `CertificateCredentialService`, és tegye megjegyzésbe a sor használó `ClientCredentialService`:

    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```

    Ez a változás lehetővé teszi a WebApp [ügyféltény] [ client-assertion] az OAuth hozzáférési tokenek beszerzése. Az ügyféltény az OAuth ügyfélkulcs nem szükséges. Másik lehetőségként tárolhatja a titkos ügyfélkulcsot a key vaultban. Azonban a key vault és ügyféltény mindkettő az ügyfél használja a tanúsítvány, hogy legyen, ha engedélyezi a key vault, célszerű ügyféltény is engedélyezi.

### <a name="update-the-user-secrets"></a>A felhasználó titkos kódok frissítése

A Megoldáskezelőben kattintson a jobb gombbal a Tailspin.Surveys.Web projektet, és válassza ki **felhasználói titkok kezelése**. A secrets.json fájlban törölje a meglévő JSON, és illessze be a következő:

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

Cserélje le a [szögletes zárójelek] bejegyzést a megfelelő értékekkel.

* `AzureAd:ClientId`: A Surveys alkalmazás ügyfél-azonosító.
* `AzureAd:ClientSecret`: Az Azure ad-ben a Surveys alkalmazás regisztrációja során létrehozott kulcs.
* `AzureAd:WebApiResourceId`: Az Alkalmazásazonosító URI-t, hogy a megadott Azure AD-ben a Surveys.WebAPI alkalmazás létrehozásakor.
* `Asymmetric:CertificateThumbprint`: A tanúsítvány ujjlenyomatát a korábban, az ügyféltanúsítvány létrehozásakor.
* `KeyVault:Name`: A kulcstartó neve.

> [!NOTE]
> `Asymmetric:ValidationRequired` hamis értéket, mert a korábban létrehozott tanúsítvány nem volt aláírva a legfelső szintű hitelesítésszolgáltató (CA). Éles környezetben állítsa be, és a legfelső szintű hitelesítésszolgáltató által aláírt tanúsítványt használjon `ValidationRequired` igaz értékre.

Mentse a frissített secrets.json fájlt.

Ezután a Megoldáskezelőben kattintson a jobb gombbal a Tailspin.Surveys.WebApi projektet, és válassza ki **felhasználói titkok kezelése**. Törölje a meglévő JSON, és illessze be a következőket:

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

Cserélje le a [szögletes zárójelek] bejegyzést, és mentse a secrets.json fájlt.

> [!NOTE]
> A webes API ügyeljen arra, hogy az ügyfél-Azonosítót használja a Surveys.WebAPI alkalmazás, nem a Surveys alkalmazás.

[**Tovább**][adfs]

<!-- links -->

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
