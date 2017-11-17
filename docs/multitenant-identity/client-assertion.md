---
title: "Ügyfél helyességi feltétel használja az Azure AD hozzáférési jogkivonatok segítségével"
description: "Hogyan használható az ügyfél helyességi feltétel az Azure AD hozzáférési jogkivonatok segítségével."
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
---
# <a name="use-client-assertion-to-get-access-tokens-from-azure-ad"></a><span data-ttu-id="4944d-103">Ügyfél helyességi feltétel használja az Azure AD hozzáférési jogkivonatok segítségével</span><span class="sxs-lookup"><span data-stu-id="4944d-103">Use client assertion to get access tokens from Azure AD</span></span>

<span data-ttu-id="4944d-104">[![GitHub](../_images/github.png) példakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="4944d-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

## <a name="background"></a><span data-ttu-id="4944d-105">Háttér</span><span class="sxs-lookup"><span data-stu-id="4944d-105">Background</span></span>
<span data-ttu-id="4944d-106">Ha hitelesítésikód-folyamata vagy hibrid folyamata a OpenID Connect, az ügyfél egy engedélyezési kódot, a hozzáférési token adatcseréihez használható.</span><span class="sxs-lookup"><span data-stu-id="4944d-106">When using authorization code flow or hybrid flow in OpenID Connect, the client exchanges an authorization code for an access token.</span></span> <span data-ttu-id="4944d-107">Ezzel a lépéssel az ügyfél számára hitelesíti magát a kiszolgáló rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="4944d-107">During this step, the client has to authenticate itself to the server.</span></span>

![Titkos ügyfélkulcs](./images/client-secret.png)

<span data-ttu-id="4944d-109">Az ügyfél hitelesítés egyik módja egy titkos ügyfélkulcsot használatával van.</span><span class="sxs-lookup"><span data-stu-id="4944d-109">One way to authenticate the client is by using a client secret.</span></span> <span data-ttu-id="4944d-110">Meg az [Dejójáték felmérések] [ Surveys] alkalmazás alapértelmezés szerint van konfigurálva.</span><span class="sxs-lookup"><span data-stu-id="4944d-110">That's how the [Tailspin Surveys][Surveys] application is configured by default.</span></span>

<span data-ttu-id="4944d-111">Íme egy példa az ügyfél a kiállító terjesztési hely, egy hozzáférési jogkivonatot kérő kérést.</span><span class="sxs-lookup"><span data-stu-id="4944d-111">Here is an example request from the client to the IDP, requesting an access token.</span></span> <span data-ttu-id="4944d-112">Megjegyzés: a `client_secret` paraméter.</span><span class="sxs-lookup"><span data-stu-id="4944d-112">Note the `client_secret` parameter.</span></span>

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_secret=i3Bf12Dn...
  &grant_type=authorization_code
  &code=PG8wJG6Y...
```

<span data-ttu-id="4944d-113">A titkos kulcsot érték csak egy karakterlánc, ezért meg kell győződnie, hogy ne szivárgás lépett fel az értéket.</span><span class="sxs-lookup"><span data-stu-id="4944d-113">The secret is just a string, so you have to make sure not to leak the value.</span></span> <span data-ttu-id="4944d-114">Az ajánlott eljárás az adatok megőrzése a titkos ügyfélkulcs verziókezelő kívül.</span><span class="sxs-lookup"><span data-stu-id="4944d-114">The best practice is to keep the client secret out of source control.</span></span> <span data-ttu-id="4944d-115">Központi telepítésekor az Azure-ba, tárolja a titkos kulcsot egy [Alkalmazásbeállítás][configure-web-app].</span><span class="sxs-lookup"><span data-stu-id="4944d-115">When you deploy to Azure, store the secret in an [app setting][configure-web-app].</span></span>

<span data-ttu-id="4944d-116">Bárki, aki hozzáféréssel rendelkezik az Azure-előfizetés megtekinthetik azonban az alkalmazás beállításaiban.</span><span class="sxs-lookup"><span data-stu-id="4944d-116">However, anyone with access to the Azure subscription can view the app settings.</span></span> <span data-ttu-id="4944d-117">További mindig fennáll a még titkos kulcsok ellenőrzi a verziókövetési rendszerrel (például az üzembe helyezési parancsfájlok), őket e-mailben megosztott és így tovább lehetőséget sem ad.</span><span class="sxs-lookup"><span data-stu-id="4944d-117">Further, there is always a temptation to check secrets into source control (e.g., in deployment scripts), share them by email, and so on.</span></span>

<span data-ttu-id="4944d-118">Használhatja a fokozott biztonság érdekében [ügyfél helyességi feltétel] helyett egy ügyfélkulcsot.</span><span class="sxs-lookup"><span data-stu-id="4944d-118">For additional security, you can use [client assertion] instead of a client secret.</span></span> <span data-ttu-id="4944d-119">Az ügyfél helyességi feltétel az ügyfél használ egy X.509 tanúsítvány igazolásához a token kérés érkezett az ügyfél.</span><span class="sxs-lookup"><span data-stu-id="4944d-119">With client assertion, the client uses an X.509 certificate to prove the token request came from the client.</span></span> <span data-ttu-id="4944d-120">Az ügyféltanúsítvány a webkiszolgálón telepítve van.</span><span class="sxs-lookup"><span data-stu-id="4944d-120">The client certificate is installed on the web server.</span></span> <span data-ttu-id="4944d-121">Általában akkor könnyebb lesz a tanúsítvány való hozzáférésének korlátozásához, mint annak érdekében, hogy senki sem véletlenül tárja fel a titkos ügyfélkulcsot.</span><span class="sxs-lookup"><span data-stu-id="4944d-121">Generally, it will be easier to restrict access to the certificate, than to ensure that nobody inadvertently reveals a client secret.</span></span> <span data-ttu-id="4944d-122">A webalkalmazásban tanúsítványok konfigurálásával kapcsolatos további információkért lásd: [tanúsítványok használata az Azure-webhelyek alkalmazásokban][using-certs-in-websites]</span><span class="sxs-lookup"><span data-stu-id="4944d-122">For more information about configuring certificates in a web app, see [Using Certificates in Azure Websites Applications][using-certs-in-websites]</span></span>

<span data-ttu-id="4944d-123">Kérelmek használatával az ügyfél helyességi feltétel a következő:</span><span class="sxs-lookup"><span data-stu-id="4944d-123">Here is a token request using client assertion:</span></span>

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

<span data-ttu-id="4944d-124">Figyelje meg, hogy a `client_secret` paraméter már nincs használatban.</span><span class="sxs-lookup"><span data-stu-id="4944d-124">Notice that the `client_secret` parameter is no longer used.</span></span> <span data-ttu-id="4944d-125">Ehelyett a `client_assertion` paraméter egy JWT jogkivonatot tartalmaz, amely alá volt írva, az ügyféltanúsítvány használatával.</span><span class="sxs-lookup"><span data-stu-id="4944d-125">Instead, the `client_assertion` parameter contains a JWT token that was signed using the client certificate.</span></span> <span data-ttu-id="4944d-126">A `client_assertion_type` paraméter határozza meg a helyességi feltétel &mdash; Ez esetben JWT jogkivonat.</span><span class="sxs-lookup"><span data-stu-id="4944d-126">The `client_assertion_type` parameter specifies the type of assertion &mdash; in this case, JWT token.</span></span> <span data-ttu-id="4944d-127">A kiszolgáló ellenőrzi a JWT jogkivonat.</span><span class="sxs-lookup"><span data-stu-id="4944d-127">The server validates the JWT token.</span></span> <span data-ttu-id="4944d-128">A JWT jogkivonat nem érvényes, ha a jogkivonatkérelem hibát ad vissza.</span><span class="sxs-lookup"><span data-stu-id="4944d-128">If the JWT token is invalid, the token request returns an error.</span></span>

> [!NOTE]
> <span data-ttu-id="4944d-129">X.509 tanúsítvány nincsenek ügyfél helyességi feltétel; csak formája a Microsoft összpontosítania, azt itt mivel Azure AD által támogatott.</span><span class="sxs-lookup"><span data-stu-id="4944d-129">X.509 certificates are not the only form of client assertion; we focus on it here because it is supported by Azure AD.</span></span>
> 
> 

<span data-ttu-id="4944d-130">Futás közben a webalkalmazás beolvassa a tanúsítványt a tanúsítványtárolóból.</span><span class="sxs-lookup"><span data-stu-id="4944d-130">At run time, the web application reads the certificate from the certificate store.</span></span> <span data-ttu-id="4944d-131">A tanúsítvány telepítenie kell a webalkalmazás ugyanazon a számítógépen.</span><span class="sxs-lookup"><span data-stu-id="4944d-131">The certificate must be installed on the same machine as the web app.</span></span>

<span data-ttu-id="4944d-132">A felmérések alkalmazás tartalmazza, amely létrehoz egy segítőosztály egy [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) , amely képes továbbadni a [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) metódus származó jogkivonat beszerzése Az Azure AD.</span><span class="sxs-lookup"><span data-stu-id="4944d-132">The Surveys application includes a helper class that creates a [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) that you can pass to the [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) method to acquire a token from Azure AD.</span></span>

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

<span data-ttu-id="4944d-133">Ügyfél helyességi feltétel felmérések alkalmazása beállításával kapcsolatos információkért lásd: [használja az Azure Key Vault alkalmazás titkos kulcsok védelme ] [ key vault].</span><span class="sxs-lookup"><span data-stu-id="4944d-133">For information about setting up client assertion in the Surveys application, see [Use Azure Key Vault to protect application secrets ][key vault].</span></span>

<span data-ttu-id="4944d-134">[**Következő**][key vault]</span><span class="sxs-lookup"><span data-stu-id="4944d-134">[**Next**][key vault]</span></span>

<!-- Links -->
[configure-web-app]: /azure/app-service-web/web-sites-configure/
[azure-management-portal]: https://portal.azure.com
[ügyfél helyességi feltétel]: https://tools.ietf.org/html/rfc7521
[key vault]: key-vault.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[using-certs-in-websites]: https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/

[sample application]: https://github.com/mspnp/multitenant-saas-guidance
