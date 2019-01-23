---
title: Ügyféltény használatával hozzáférési tokenek beszerzése az Azure ad-ből
description: Hogyan ügyféltény hozzáférési tokenek beszerzése az Azure ad-ből.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: adfs
pnp.series.next: key-vault
ms.openlocfilehash: a1ea65ca8f6b98932f0c42a1b029efeca4d50710
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54482425"
---
# <a name="use-client-assertion-to-get-access-tokens-from-azure-ad"></a>Ügyféltény használatával hozzáférési tokenek beszerzése az Azure ad-ből

[![GitHub](../_images/github.png) Mintakód][sample application]

## <a name="background"></a>Háttér

OpenID Connect hitelesítési kódfolyamat vagy hibrid folyamatot használ, amikor az ügyfél hozzáférési jogkivonatot engedélyezési kódjának adatcseréihez használható. Ezzel a lépéssel az ügyfél rendelkezik, hitelesítse magát a kiszolgálót.

![Titkos ügyfélkód](./images/client-secret.png)

Az ügyfél hitelesítésére egy úgy, hogy egy ügyfél titkos kulcs használatával. Hogy a [Tailspin Surveys] [ Surveys] alkalmazás alapértelmezés szerint van konfigurálva.

Íme egy példa az ügyfél az Identitásszolgáltató hozzáférési jogkivonatot kér kérést. Megjegyzés: a `client_secret` paraméter.

```http
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_secret=i3Bf12Dn...
  &grant_type=authorization_code
  &code=PG8wJG6Y...
```

A titkos kulcs csak egy karakterlánc,, ezért ügyeljen arra, hogy nem szivárogtatnak ki az értéket kell. Az ajánlott eljárás, hogy a titkos ügyfélkulcsot verziókövetés tartományon kívül. Az Azure-ba történő telepítésekor tárolhatja a titkos kulcsot egy [Alkalmazásbeállítás][configure-web-app].

Azonban az Azure-előfizetés segítségével bárki megtekintheti az alkalmazásbeállításokat. További mindig van egy kísértésnek, ellenőrizze a titkos kulcsok (például az üzembehelyezési szkriptek) forrásvezérlőben, e-mailben megosztani őket, és így tovább.

A fokozott biztonság érdekében használhat [ügyféltény] ügyfélkódot helyett. A ügyféltény, az ügyfél használatával egy X.509 tanúsítvány igazolja, hogy az ügyfél származik a jogkivonat kérése. Az ügyféltanúsítvány a webkiszolgálón telepítve van. Általában a könnyebb lesz a tanúsítvány való hozzáférés korlátozásához, mint annak érdekében, hogy senki sem véletlenül tárja fel az ügyfél titkos kulcs. Webalkalmazás-ban történő tanúsítványkonfigurálással kapcsolatos további információkért lásd: [tanúsítványok használata az Azure Websites-alkalmazások][using-certs-in-websites]

A következő kérés ügyféltény használatával:

```http
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
  &client_assertion=eyJhbGci...
  &grant_type=authorization_code
  &code= PG8wJG6Y...
```

Figyelje meg, hogy a `client_secret` paraméter nincs többé használatban. Ehelyett a `client_assertion` paraméter tartalmazza a JWT jogkivonat, amely az ügyféltanúsítvány használatával lett aláírva. A `client_assertion_type` paraméter meghatározza a helyességi feltétel &mdash; az ebben az esetben JWT jogkivonat. A kiszolgáló ellenőrzi a JWT jogkivonat. A JWT jogkivonat nem érvényes, ha a jogkivonat kérése hibát ad vissza.

> [!NOTE]
> X.509-tanúsítványokat amelyek nem csak formájában ügyféltény; fogunk összpontosítani, itt, mert az Azure AD által támogatott.

Futási időben a webalkalmazás beolvassa a tanúsítványt a tanúsítványtárolóból. A tanúsítvány telepítenie kell a webalkalmazás ugyanazon a gépen.

A Surveys alkalmazás tartalmaz, amely létrehoz egy segítőosztály egy [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) , amely átadható a [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) származó jogkivonat beszerzésére metódus Azure ad-ben.

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

A Surveys alkalmazás ügyféltény beállításával kapcsolatos további információkért lásd: [Azure Key Vault használata a titkos alkalmazáskulcsok védelme ] [ key vault].

[**Tovább**][key vault]

<!-- links -->

[configure-web-app]: /azure/app-service-web/web-sites-configure/
[azure-management-portal]: https://portal.azure.com
[ügyféltény]: https://tools.ietf.org/html/rfc7521
[key vault]: key-vault.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[using-certs-in-websites]: https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/

[sample application]: https://github.com/mspnp/multitenant-saas-guidance
