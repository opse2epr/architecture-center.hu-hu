---
title: Key Vault segítségével alkalmazás titkos kulcsok védelme
description: A Key Vault szolgáltatás használata az alkalmazás titkos kulcsok tárolására
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
ms.openlocfilehash: 45d1564c255f2450f68c5e92ebe0d7de0c40ae31
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="use-azure-key-vault-to-protect-application-secrets"></a>Az Azure Key Vault segítségével alkalmazás titkos kulcsok védelme

[![GitHub](../_images/github.png) példakód][sample application]

Esetében gyakori, hogy bizalmas és védeni kell, például alkalmazás beállításait:

* Adatbázis-kapcsolati karakterláncok
* Jelszavak
* Titkosítási kulcsok

Biztonsági szempontból ajánlott a verziókövetési rendszerrel ezeknek a kulcsoknak soha nem tárolja. Annak érdekében, hogy nyilvánosságra kerüljenek túl könnyű &mdash; akkor is, ha a forráskódraktárban személyes. És azt nem szinte megakadályozza a titkos kulcsok a főkönyvi nyilvános. A nagyobb projektek mely fejlesztők korlátozni lehet, hogy szeretné, és operátorok férhetnek hozzá a termelési titkos kulcsok. (A teszt- vagy környezetekben található beállítások eltérnek.)

A titkos adatok tárolására egy biztonságosabb beállítás [Azure Key Vault][KeyVault]. Key Vault egy a felhőben üzemeltetett szolgáltatás kriptográfiai kulcsokat és más titkos kulcsok kezeléséhez. Ez a cikk bemutatja, hogyan tárolni a konfigurációs beállításokat az alkalmazást a Key Vault használatával.

Az a [Dejójáték felmérések] [ Surveys] alkalmazás, a következő beállítások érhetők titkos:

* Az adatbázis-kapcsolati karakterlánc.
* A Redis kapcsolódási karakterlánc.
* A webes alkalmazás titkos ügyfélkódot.

A felmérések alkalmazás betölt a konfigurációs beállításokat a következő helyről:

* A appsettings.json fájl
* A [felhasználói titkos kulcsok tárolási] [ user-secrets] (fejlesztőkörnyezet; csak tesztelési célra)
* Az üzemeltetési környezet (az Azure web apps Alkalmazásbeállítások)
* Key Vault (Ha engedélyezve van)

Minden egyes ezeket a felülbírálásokat az előzőre, így a Key Vault tárolt beállítások elsőbbséget.

> [!NOTE]
> Alapértelmezés szerint le van tiltva a Key Vault konfigurációszolgáltatót. Nincs szükség az alkalmazás helyi futtatásához. Éles környezetben azt szeretné engedélyezni.

Indításkor az alkalmazás beállítást olvassa ki minden regisztrált konfigurációszolgáltatót, és egy szigorú típusmegadású beállítások objektum feltöltéséhez használja őket. További információkért lásd: [lehetőségek használata és konfigurációs objektumok][options].

## <a name="setting-up-key-vault-in-the-surveys-app"></a>Key Vault a felmérések alkalmazás beállítása
Előfeltételek:

* Telepítse a [Azure Resource Manager parancsmagjainak][azure-rm-cmdlets].
* Konfigurálja az felmérések alkalmazást, a [futtassa a felmérések alkalmazást][readme].

Magas szintű lépéseket:

1. Állítson be egy rendszergazdai jogú felhasználót a bérlőben.
2. Állítson be egy ügyféltanúsítványt.
3. Hozzon létre egy kulcstartót.
4. Konfigurációs beállítások hozzáadása a kulcstartót.
5. Állítsa vissza a kódot, amely lehetővé teszi, hogy a kulcstároló.
6. Frissítse az alkalmazás felhasználói titkos kulcsok.

### <a name="set-up-an-admin-user"></a>Állítson be egy rendszergazdai jogú felhasználó
> [!NOTE]
> Hozzon létre egy kulcstartót, kezelheti az Azure-előfizetéshez fiókot kell használnia. Emellett bármely alkalmazás, amely a key vault olvasni engedélyeznie kell regisztrálni az ugyanannak a bérlőnek, mint ez a fiók.
> 
> 

Ebben a lépésben teheti meg arról, hogy is létrehozott egy kulcstartót, miközben a bérlőtől felhasználóként van bejelentkezve az ahol az felmérések alkalmazás a regisztrációt.

Hozzon létre egy rendszergazda felhasználó, ahol a felmérések alkalmazás regisztrálva van az Azure AD-bérlő belül.

1. Jelentkezzen be a [Azure-portálon][azure-portal].
2. Válassza ki az Azure AD-bérlő, ahol az alkalmazás regisztrálva van.
3. Kattintson a **további szolgáltatás** > **biztonság + identitás szakaszában** > **Azure Active Directory** > **felhasználók és csoportok**   >  **Minden felhasználó**.
4. A portál felső részén kattintson **új felhasználó**.
5. Töltse ki a mezőket, és rendelje hozzá a felhasználót a **globális rendszergazda** címtár szerepkörrel.
6. Kattintson a **Create** (Létrehozás) gombra.

![Globális rendszergazdai jogú felhasználó](./images/running-the-app/global-admin-user.png)

Most már az előfizetés tulajdonosa a felhasználóhoz rendelni.

1. A központ menüben válassza ki a **előfizetések**.

    ![](./images/running-the-app/subscriptions.png)

2. Válassza ki az előfizetést, amelyet a rendszergazda eléréséhez.
3. Az előfizetés panelen válassza ki a **hozzáférés-vezérlés (IAM)**.
4. Kattintson az **Add** (Hozzáadás) parancsra.
4. A **szerepkör**, jelölje be **tulajdonos**.
5. Írja be a felhasználó hozzá szeretne adni a tulajdonos e-mail címe.
6. Válassza ki a felhasználót, és kattintson a **mentése**.

### <a name="set-up-a-client-certificate"></a>Ügyfél-tanúsítvány beállítása
1. A PowerShell-parancsprogrammal [/Scripts/Setup-KeyVault.ps1] [ Setup-KeyVault] az alábbiak szerint:
   
    ```
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    Az a `Subject` paraméter, adja meg a nevét, például a "surveysapp". A parancsfájl létrehoz egy önaláírt tanúsítványt, és tárolja a "jelenlegi felhasználó/Personal" tanúsítványtárolójában. Parancsfájl eredménye a JSON-töredéket. Másolja ezt az értéket.

2. Az a [Azure-portálon][azure-portal], lépjen a mappába, ahol a a felmérések alkalmazás regisztrálva, a fiók kiválasztja a portál jobb felső sarkában.

3. Válassza ki **az Azure Active Directory** > **App regisztrációk** > felmérések

4.  Kattintson a **Manifest** , majd **szerkesztése**.

5.  Illessze be a parancsfájlt a kimenetét a `keyCredentials` tulajdonság. Ennek a következőképpen kell kinéznie:
        
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

7. Ismételje meg a 3-6 ugyanazon JSON töredék hozzáadása a Web API (Surveys.WebAPI) az alkalmazás jegyzékében.

8. A PowerShell-ablakban futtassa a következő parancsot a tanúsítvány ujjlenyomatát.
   
    ```
    certutil -store -user my [subject]
    ```
    
    A `[subject]`, a tulajdonos a PowerShell-parancsfájl a megadott értéket használja. Az ujjlenyomat megtalálható-e a "Cert Hash(sha1)". Másolja ezt az értéket. Az ujjlenyomat később fogja használni.

### <a name="create-a-key-vault"></a>Kulcstartó létrehozása
1. A PowerShell-parancsprogrammal [/Scripts/Setup-KeyVault.ps1] [ Setup-KeyVault] az alábbiak szerint:
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```
   
    Ha a hitelesítő adatok megadását kéri, jelentkezzen be az Azure AD-felhasználó, korábban létrehozott. A parancsfájl létrehoz egy új erőforráscsoportot, és egy új kulcstartó adott erőforráscsoporton belül. 
   
2. Futtassa újra SetupKeyVault.ps az alábbiak szerint:
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<Surveys app id>>", "<<Surveys.WebAPI app ID>>")
    ```
   
    Állítsa be a következő paraméterértékeket:
   
       * kulcstároló neve = a neve, mint a kulcstároló az előző lépésben.
       * Alkalmazásazonosító felmérések = az Alkalmazásazonosítót a felmérések webalkalmazáshoz.
       * Surveys.WebApi app ID = az Alkalmazásazonosítót a Surveys.WebAPI alkalmazáshoz.
         
    Példa:
     
    ```
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
    ```
    
    Ezt a parancsfájlt a webes alkalmazás és a titkos kulcsok lekérése a kulcstartót webes API engedélyezi. Lásd: [Ismerkedés az Azure Key Vault](/azure/key-vault/key-vault-get-started/) további információt.

### <a name="add-configuration-settings-to-your-key-vault"></a>Konfigurációs beállítások hozzáadása a kulcstartót
1. Futtassa a következőképpen SetupKeyVault.ps::
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Redis--Configuration -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true" 
    ```
    Ha
   
   * kulcstároló neve = a neve, mint a kulcstároló az előző lépésben.
   * DNS-név redis = a Redis cache példány DNS-neve.
   * Redis hozzáférési kulcsot a hozzáférési kulcsot a Redis cache példány =.
     
2. Ezen a ponton akkor érdemes annak megállapítására, hogy sikeresen tárolta a titkos kulcsok számára kulcstároló. Futtassa a következő PowerShell-parancsot:
   
    ```
    Get-AzureKeyVaultSecret <<key vault name>> Redis--Configuration | Select-Object *
    ```

3. Futtassa újra az adatbázis-kapcsolati karakterlánc hozzáadása a SetupKeyVault.ps:
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Data--SurveysConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```
   
    Ha `<<DB connection string>>` van az adatbázis-kapcsolati karakterlánc értékét.
   
    A helyi adatbázis tesztelték, másolja a kapcsolati karakterláncot a Tailspin.Surveys.Web/appsettings.json fájlból. Ha így tesz, ügyeljen arra, hogy módosítsa a dupla fordított perjel ('\\\\") az egyetlen perjellel. A kettős fordított karakteréhez escape-karakter, a JSON-fájlban.
   
    Példa:
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName Data--SurveysConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true" 
    ```

### <a name="uncomment-the-code-that-enables-key-vault"></a>Állítsa vissza a kódot, amely lehetővé teszi, hogy a Key Vault
1. Nyissa meg a Tailspin.Surveys megoldást.
2. A Tailspin.Surveys.Web/Startup.cs keresse meg a következő kódrészletet, és állítsa vissza azt.
   
    ```csharp
    //var config = builder.Build();
    //builder.AddAzureKeyVault(
    //    $"https://{config["KeyVault:Name"]}.vault.azure.net/",
    //    config["AzureAd:ClientId"],
    //    config["AzureAd:ClientSecret"]);
    ```
3. Tailspin.Surveys.Web/Startup.cs, keresse meg a kódot, amely regisztrálja a `ICredentialService`. Állítsa vissza a sor által használt `CertificateCredentialService`, és a sor által használt megjegyzésbe `ClientCredentialService`:
   
    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```
   
    A módosítás lehetővé teszi a webalkalmazás használandó [ügyfél helyességi feltétel] [ client-assertion] lekérni az OAuth jogkivonatot. Az ügyfél helyességi feltétel az OAuth ügyfélkulcs nem szükséges. Azt is megteheti a titkos ügyfélkulcs tárolhatja a key vaultban. Azonban kulcstartó és mindkettő használja, hogy egy ügyfél ügyfél helyességi feltétel tanúsítvány, ha engedélyezi a kulcstároló, hogy célszerű engedélyezni, valamint az ügyfél állítás.

### <a name="update-the-user-secrets"></a>A felhasználó titkos kulcsok frissítése
A Megoldáskezelőben kattintson a jobb gombbal a Tailspin.Surveys.Web projektet, és válassza ki **felhasználói kulcsok kezelése**. A secrets.json fájlban törölje a meglévő JSON, és illessze be a következőket:

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

A bejegyzések [szögletes zárójelbe] cserélje le a megfelelő értékeket.

* `AzureAd:ClientId`: Az a felmérések alkalmazás ügyfél-azonosító.
* `AzureAd:ClientSecret`: A kulcs az Azure ad-ben a felmérések alkalmazás regisztrálásakor létrehozott.
* `AzureAd:WebApiResourceId`: A App ID URI Azure AD-ben a Surveys.WebAPI alkalmazás létrehozásakor megadott.
* `Asymmetric:CertificateThumbprint`: A tanúsítvány ujjlenyomata portáltól korábban, az ügyfél-tanúsítvány létrehozásakor.
* `KeyVault:Name`: A kulcstartót a neve.

> [!NOTE]
> `Asymmetric:ValidationRequired`hamis, mert a korábban létrehozott tanúsítvány nem volt által aláírt egy legfelső szintű hitelesítésszolgáltatót (CA). Éles, állítsa be, és a legfelső szintű hitelesítésszolgáltató által aláírt tanúsítványt használjon `ValidationRequired` igaz értékre.
> 
> 

Mentse a frissített secrets.json fájlt.

Ezután a Megoldáskezelőben kattintson a jobb gombbal a Tailspin.Surveys.WebApi projektet, és válassza ki **felhasználói kulcsok kezelése**. Törölje a meglévő JSON, és illessze be a következő:

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

Cserélje le a [szögletes zárójelbe] bejegyzések, és mentse a secrets.json fájlt.

> [!NOTE]
> A webes API-t győződjön meg arról, az ügyfél-azonosító használata a Surveys.WebAPI alkalmazáshoz, nem a felmérések alkalmazás.
> 
> 

[**Következő**][adfs]

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
