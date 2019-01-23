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
# <a name="use-client-assertion-to-get-access-tokens-from-azure-ad"></a><span data-ttu-id="d59a9-103">Ügyféltény használatával hozzáférési tokenek beszerzése az Azure ad-ből</span><span class="sxs-lookup"><span data-stu-id="d59a9-103">Use client assertion to get access tokens from Azure AD</span></span>

<span data-ttu-id="d59a9-104">[![GitHub](../_images/github.png) Mintakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="d59a9-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

## <a name="background"></a><span data-ttu-id="d59a9-105">Háttér</span><span class="sxs-lookup"><span data-stu-id="d59a9-105">Background</span></span>

<span data-ttu-id="d59a9-106">OpenID Connect hitelesítési kódfolyamat vagy hibrid folyamatot használ, amikor az ügyfél hozzáférési jogkivonatot engedélyezési kódjának adatcseréihez használható.</span><span class="sxs-lookup"><span data-stu-id="d59a9-106">When using authorization code flow or hybrid flow in OpenID Connect, the client exchanges an authorization code for an access token.</span></span> <span data-ttu-id="d59a9-107">Ezzel a lépéssel az ügyfél rendelkezik, hitelesítse magát a kiszolgálót.</span><span class="sxs-lookup"><span data-stu-id="d59a9-107">During this step, the client has to authenticate itself to the server.</span></span>

![Titkos ügyfélkód](./images/client-secret.png)

<span data-ttu-id="d59a9-109">Az ügyfél hitelesítésére egy úgy, hogy egy ügyfél titkos kulcs használatával.</span><span class="sxs-lookup"><span data-stu-id="d59a9-109">One way to authenticate the client is by using a client secret.</span></span> <span data-ttu-id="d59a9-110">Hogy a [Tailspin Surveys] [ Surveys] alkalmazás alapértelmezés szerint van konfigurálva.</span><span class="sxs-lookup"><span data-stu-id="d59a9-110">That's how the [Tailspin Surveys][Surveys] application is configured by default.</span></span>

<span data-ttu-id="d59a9-111">Íme egy példa az ügyfél az Identitásszolgáltató hozzáférési jogkivonatot kér kérést.</span><span class="sxs-lookup"><span data-stu-id="d59a9-111">Here is an example request from the client to the IDP, requesting an access token.</span></span> <span data-ttu-id="d59a9-112">Megjegyzés: a `client_secret` paraméter.</span><span class="sxs-lookup"><span data-stu-id="d59a9-112">Note the `client_secret` parameter.</span></span>

```http
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_secret=i3Bf12Dn...
  &grant_type=authorization_code
  &code=PG8wJG6Y...
```

<span data-ttu-id="d59a9-113">A titkos kulcs csak egy karakterlánc,, ezért ügyeljen arra, hogy nem szivárogtatnak ki az értéket kell.</span><span class="sxs-lookup"><span data-stu-id="d59a9-113">The secret is just a string, so you have to make sure not to leak the value.</span></span> <span data-ttu-id="d59a9-114">Az ajánlott eljárás, hogy a titkos ügyfélkulcsot verziókövetés tartományon kívül.</span><span class="sxs-lookup"><span data-stu-id="d59a9-114">The best practice is to keep the client secret out of source control.</span></span> <span data-ttu-id="d59a9-115">Az Azure-ba történő telepítésekor tárolhatja a titkos kulcsot egy [Alkalmazásbeállítás][configure-web-app].</span><span class="sxs-lookup"><span data-stu-id="d59a9-115">When you deploy to Azure, store the secret in an [app setting][configure-web-app].</span></span>

<span data-ttu-id="d59a9-116">Azonban az Azure-előfizetés segítségével bárki megtekintheti az alkalmazásbeállításokat.</span><span class="sxs-lookup"><span data-stu-id="d59a9-116">However, anyone with access to the Azure subscription can view the app settings.</span></span> <span data-ttu-id="d59a9-117">További mindig van egy kísértésnek, ellenőrizze a titkos kulcsok (például az üzembehelyezési szkriptek) forrásvezérlőben, e-mailben megosztani őket, és így tovább.</span><span class="sxs-lookup"><span data-stu-id="d59a9-117">Further, there is always a temptation to check secrets into source control (e.g., in deployment scripts), share them by email, and so on.</span></span>

<span data-ttu-id="d59a9-118">A fokozott biztonság érdekében használhat [ügyféltény] ügyfélkódot helyett.</span><span class="sxs-lookup"><span data-stu-id="d59a9-118">For additional security, you can use [client assertion] instead of a client secret.</span></span> <span data-ttu-id="d59a9-119">A ügyféltény, az ügyfél használatával egy X.509 tanúsítvány igazolja, hogy az ügyfél származik a jogkivonat kérése.</span><span class="sxs-lookup"><span data-stu-id="d59a9-119">With client assertion, the client uses an X.509 certificate to prove the token request came from the client.</span></span> <span data-ttu-id="d59a9-120">Az ügyféltanúsítvány a webkiszolgálón telepítve van.</span><span class="sxs-lookup"><span data-stu-id="d59a9-120">The client certificate is installed on the web server.</span></span> <span data-ttu-id="d59a9-121">Általában a könnyebb lesz a tanúsítvány való hozzáférés korlátozásához, mint annak érdekében, hogy senki sem véletlenül tárja fel az ügyfél titkos kulcs.</span><span class="sxs-lookup"><span data-stu-id="d59a9-121">Generally, it will be easier to restrict access to the certificate, than to ensure that nobody inadvertently reveals a client secret.</span></span> <span data-ttu-id="d59a9-122">Webalkalmazás-ban történő tanúsítványkonfigurálással kapcsolatos további információkért lásd: [tanúsítványok használata az Azure Websites-alkalmazások][using-certs-in-websites]</span><span class="sxs-lookup"><span data-stu-id="d59a9-122">For more information about configuring certificates in a web app, see [Using Certificates in Azure Websites Applications][using-certs-in-websites]</span></span>

<span data-ttu-id="d59a9-123">A következő kérés ügyféltény használatával:</span><span class="sxs-lookup"><span data-stu-id="d59a9-123">Here is a token request using client assertion:</span></span>

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

<span data-ttu-id="d59a9-124">Figyelje meg, hogy a `client_secret` paraméter nincs többé használatban.</span><span class="sxs-lookup"><span data-stu-id="d59a9-124">Notice that the `client_secret` parameter is no longer used.</span></span> <span data-ttu-id="d59a9-125">Ehelyett a `client_assertion` paraméter tartalmazza a JWT jogkivonat, amely az ügyféltanúsítvány használatával lett aláírva.</span><span class="sxs-lookup"><span data-stu-id="d59a9-125">Instead, the `client_assertion` parameter contains a JWT token that was signed using the client certificate.</span></span> <span data-ttu-id="d59a9-126">A `client_assertion_type` paraméter meghatározza a helyességi feltétel &mdash; az ebben az esetben JWT jogkivonat.</span><span class="sxs-lookup"><span data-stu-id="d59a9-126">The `client_assertion_type` parameter specifies the type of assertion &mdash; in this case, JWT token.</span></span> <span data-ttu-id="d59a9-127">A kiszolgáló ellenőrzi a JWT jogkivonat.</span><span class="sxs-lookup"><span data-stu-id="d59a9-127">The server validates the JWT token.</span></span> <span data-ttu-id="d59a9-128">A JWT jogkivonat nem érvényes, ha a jogkivonat kérése hibát ad vissza.</span><span class="sxs-lookup"><span data-stu-id="d59a9-128">If the JWT token is invalid, the token request returns an error.</span></span>

> [!NOTE]
> <span data-ttu-id="d59a9-129">X.509-tanúsítványokat amelyek nem csak formájában ügyféltény; fogunk összpontosítani, itt, mert az Azure AD által támogatott.</span><span class="sxs-lookup"><span data-stu-id="d59a9-129">X.509 certificates are not the only form of client assertion; we focus on it here because it is supported by Azure AD.</span></span>

<span data-ttu-id="d59a9-130">Futási időben a webalkalmazás beolvassa a tanúsítványt a tanúsítványtárolóból.</span><span class="sxs-lookup"><span data-stu-id="d59a9-130">At run time, the web application reads the certificate from the certificate store.</span></span> <span data-ttu-id="d59a9-131">A tanúsítvány telepítenie kell a webalkalmazás ugyanazon a gépen.</span><span class="sxs-lookup"><span data-stu-id="d59a9-131">The certificate must be installed on the same machine as the web app.</span></span>

<span data-ttu-id="d59a9-132">A Surveys alkalmazás tartalmaz, amely létrehoz egy segítőosztály egy [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) , amely átadható a [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) származó jogkivonat beszerzésére metódus Azure ad-ben.</span><span class="sxs-lookup"><span data-stu-id="d59a9-132">The Surveys application includes a helper class that creates a [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) that you can pass to the [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) method to acquire a token from Azure AD.</span></span>

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

<span data-ttu-id="d59a9-133">A Surveys alkalmazás ügyféltény beállításával kapcsolatos további információkért lásd: [Azure Key Vault használata a titkos alkalmazáskulcsok védelme ] [ key vault].</span><span class="sxs-lookup"><span data-stu-id="d59a9-133">For information about setting up client assertion in the Surveys application, see [Use Azure Key Vault to protect application secrets ][key vault].</span></span>

<span data-ttu-id="d59a9-134">[**Tovább**][key vault]</span><span class="sxs-lookup"><span data-stu-id="d59a9-134">[**Next**][key vault]</span></span>

<!-- links -->

[configure-web-app]: /azure/app-service-web/web-sites-configure/
[azure-management-portal]: https://portal.azure.com
[ügyféltény]: https://tools.ietf.org/html/rfc7521
[client assertion]: https://tools.ietf.org/html/rfc7521
[key vault]: key-vault.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[using-certs-in-websites]: https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/

[sample application]: https://github.com/mspnp/multitenant-saas-guidance
