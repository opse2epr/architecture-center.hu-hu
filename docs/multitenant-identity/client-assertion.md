---
title: Ügyfél helyességi feltétel használja az Azure AD hozzáférési jogkivonatok segítségével
description: Hogyan használható az ügyfél helyességi feltétel az Azure AD hozzáférési jogkivonatok segítségével.
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: adfs
pnp.series.next: key-vault
ms.openlocfilehash: 9fe1ee2ec5a540edc41c3a310476507f8d862f0c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540281"
---
# <a name="use-client-assertion-to-get-access-tokens-from-azure-ad"></a>Ügyfél helyességi feltétel használja az Azure AD hozzáférési jogkivonatok segítségével

[![GitHub](../_images/github.png) példakód][sample application]

## <a name="background"></a>Háttér
Ha hitelesítésikód-folyamata vagy hibrid folyamata a OpenID Connect, az ügyfél egy engedélyezési kódot, a hozzáférési token adatcseréihez használható. Ezzel a lépéssel az ügyfél számára hitelesíti magát a kiszolgáló rendelkezik.

![Titkos ügyfélkulcs](./images/client-secret.png)

Az ügyfél hitelesítés egyik módja egy titkos ügyfélkulcsot használatával van. Meg az [Dejójáték felmérések] [ Surveys] alkalmazás alapértelmezés szerint van konfigurálva.

Íme egy példa az ügyfél a kiállító terjesztési hely, egy hozzáférési jogkivonatot kérő kérést. Megjegyzés: a `client_secret` paraméter.

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_secret=i3Bf12Dn...
  &grant_type=authorization_code
  &code=PG8wJG6Y...
```

A titkos kulcsot érték csak egy karakterlánc, ezért meg kell győződnie, hogy ne szivárgás lépett fel az értéket. Az ajánlott eljárás az adatok megőrzése a titkos ügyfélkulcs verziókezelő kívül. Központi telepítésekor az Azure-ba, tárolja a titkos kulcsot egy [Alkalmazásbeállítás][configure-web-app].

Bárki, aki hozzáféréssel rendelkezik az Azure-előfizetés megtekinthetik azonban az alkalmazás beállításaiban. További mindig fennáll a még titkos kulcsok ellenőrzi a verziókövetési rendszerrel (például az üzembe helyezési parancsfájlok), őket e-mailben megosztott és így tovább lehetőséget sem ad.

Használhatja a fokozott biztonság érdekében [ügyfél helyességi feltétel] helyett egy ügyfélkulcsot. Az ügyfél helyességi feltétel az ügyfél használ egy X.509 tanúsítvány igazolásához a token kérés érkezett az ügyfél. Az ügyféltanúsítvány a webkiszolgálón telepítve van. Általában akkor könnyebb lesz a tanúsítvány való hozzáférésének korlátozásához, mint annak érdekében, hogy senki sem véletlenül tárja fel a titkos ügyfélkulcsot. A webalkalmazásban tanúsítványok konfigurálásával kapcsolatos további információkért lásd: [tanúsítványok használata az Azure-webhelyek alkalmazásokban][using-certs-in-websites]

Kérelmek használatával az ügyfél helyességi feltétel a következő:

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
  &client_assertion=eyJhbGci...
  &grant_type=authorization_code
  &code= PG8wJG6Y...
```

Figyelje meg, hogy a `client_secret` paraméter már nincs használatban. Ehelyett a `client_assertion` paraméter egy JWT jogkivonatot tartalmaz, amely alá volt írva, az ügyféltanúsítvány használatával. A `client_assertion_type` paraméter határozza meg a helyességi feltétel &mdash; Ez esetben JWT jogkivonat. A kiszolgáló ellenőrzi a JWT jogkivonat. A JWT jogkivonat nem érvényes, ha a jogkivonatkérelem hibát ad vissza.

> [!NOTE]
> X.509 tanúsítvány nincsenek ügyfél helyességi feltétel; csak formája a Microsoft összpontosítania, azt itt mivel Azure AD által támogatott.
> 
> 

Futás közben a webalkalmazás beolvassa a tanúsítványt a tanúsítványtárolóból. A tanúsítvány telepítenie kell a webalkalmazás ugyanazon a számítógépen.

A felmérések alkalmazás tartalmazza, amely létrehoz egy segítőosztály egy [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) , amely képes továbbadni a [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) metódus származó jogkivonat beszerzése Az Azure AD.

```csharp
public class CertificateCredentialService : ICredentialService
{
    private Lazy<Task<AdalCredential>> _credential;

    public CertificateCredentialService(IOptions<ConfigurationOptions> options)
    {
        var aadOptions = options.Value?.AzureAd;
        _credential = new Lazy<Task<AdalCredential>>(() =>
        {
            X509Certificate2 cert = CertificateUtility.FindCertificateByThumbprint(
                aadOptions.Asymmetric.StoreName,
                aadOptions.Asymmetric.StoreLocation,
                aadOptions.Asymmetric.CertificateThumbprint,
                aadOptions.Asymmetric.ValidationRequired);
            string password = null;
            var certBytes = CertificateUtility.ExportCertificateWithPrivateKey(cert, out password);
            return Task.FromResult(new AdalCredential(new ClientAssertionCertificate(aadOptions.ClientId, new X509Certificate2(certBytes, password))));
        });
    }

    public async Task<AdalCredential> GetCredentialsAsync()
    {
        return await _credential.Value;
    }
}
```

Ügyfél helyességi feltétel felmérések alkalmazása beállításával kapcsolatos információkért lásd: [használja az Azure Key Vault alkalmazás titkos kulcsok védelme ] [ key vault].

[**Következő**][key vault]

<!-- Links -->
[configure-web-app]: /azure/app-service-web/web-sites-configure/
[azure-management-portal]: https://portal.azure.com
[ügyfél helyességi feltétel]: https://tools.ietf.org/html/rfc7521
[key vault]: key-vault.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[using-certs-in-websites]: https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/

[sample application]: https://github.com/mspnp/multitenant-saas-guidance
