---
title: Összevonás az ügyfél AD FS szolgáltatásával
description: Hogyan az ügyfél-val összevont a az AD FS egy több-bérlős alkalmazásban.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: token-cache
pnp.series.next: client-assertion
ms.openlocfilehash: b129d2de8ea3f19df8aa38cc4660885f22f9d9c6
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299638"
---
# <a name="federate-with-a-customers-ad-fs"></a>Összevonás az ügyfél AD FS szolgáltatásával

Ez a cikk bemutatja, hogyan egy több-bérlős SaaS-alkalmazáshoz is támogatják a hitelesítést az Active Directory összevonási szolgáltatások (AD FS), annak érdekében, hogy egy ügyfél AD FS vonhat össze.

## <a name="overview"></a>Áttekintés

Az Azure Active Directory (Azure AD) megkönnyíti a felhasználók az Azure AD-bérlő, többek között az Office 365 és Dynamics CRM Online ügyfelek. De mi a helyzet ügyfeleink, akik a helyszíni Active Directory a vállalati intraneten?

Az egyik lehetőség van, ezek az ügyfelek számára, hogy szinkronizálja a helyszíni AD és az Azure AD használatával [Azure AD Connect]. Egyes ügyfeleink azonban nem használható ezzel a módszerrel a vállalati informatikai házirend miatt vagy egyéb okból kifolyólag lehet. Ebben az esetben egy másik lehetőség, hogy az Active Directory összevonási szolgáltatások (AD FS) keresztül vonhat össze.

Ebben a forgatókönyvben engedélyezése:

* Az ügyfél rendelkezik egy internetkapcsolattal rendelkező AD FS-farm.
* Az SaaS-szolgáltatónak a saját AD FS-farmot helyez üzembe.
* Az ügyfél és az SaaS-szolgáltató be kell állítania a [összevonási megbízhatósági kapcsolatot]. Ez a manuális folyamat.

A megbízhatósági kapcsolatban van a három fő szerepkörök:

* Az ügyfél az AD FS a a [fiókpartner]felelős az ügyfél a felhasználók hitelesítését, ad, és a felhasználói jogcímek biztonsági jogkivonatokat hoz létre.
* Az SaaS-szolgáltatónak az AD FS a a [erőforráspartner], amely a fiókpartner megbízik, és megkapja a felhasználói jogcímeket.
* Az alkalmazás az SaaS-szolgáltatónak az AD FS megbízható függő entitásonkénti (RP) van konfigurálva.
  
  ![összevonási megbízhatósági kapcsolat](./images/federation-trust.png)

> [!NOTE]
> Ez a cikk feltételezzük használja az alkalmazás OpenID Connect hitelesítési protokoll. Egy másik lehetőség, hogy a WS-Federation használja.
>
> OpenID Connect az SaaS-szolgáltatónak az AD FS 2016, Windows Server 2016-ban futó kell használnia. Az AD FS 3.0 nem támogatja az OpenID Connect.
>
> WS-Federation-a-beépített támogatása ASP.NET Core nem tartalmazza.
>
>

Az ASP.NET 4 WS-Federation használatának példájáért lásd a [active-directory-dotnet-webapp-wsfederation minta][active-directory-dotnet-webapp-wsfederation].

## <a name="authentication-flow"></a>A hitelesítési folyamatból

1. Amikor a felhasználó a "bejelentkezés" gombra kattint, az alkalmazás átirányítja a SaaS-szolgáltató az AD FS OpenID Connect végpontja.
2. A felhasználó megadja a szervezeti felhasználó nevét ("`alice@corp.contoso.com`"). Az AD FS hitelesítőtartomány átirányítása az ügyfél AD FS, ahol a felhasználó megadja a hitelesítő adatokat használja.
3. Az ügyfél AD FS elküldi a felhasználói jogcímeket, az SaaS-szolgáltatónak AD FS-ben a WF-összevonás (vagy SAML) használatával.
4. Jogcímek folyamatot az AD FS az alkalmazáshoz, OpenID Connect használatával. Ehhez a WS-Federation protokollt átállás.

## <a name="limitations"></a>Korlátozások

Alapértelmezés szerint a függő gyártótól származó alkalmazás fogadja az id_token, az alábbi táblázatban látható a rendelkezésre álló jogcímek készletét rögzített. Az AD FS 2016 az OpenID Connect forgatókönyvekben id_token szabhatja testre. További információkért lásd: [egyéni azonosító-jogkivonatokat az AD FS](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016).

| Jogcím | Leírás |
| --- | --- |
| AUD |Célközönség. Az alkalmazás, amelynek a jogcímek adtak ki. |
| AuthenticationInstant |[Azonnali hitelesítés]. Melyik hitelesítési ideje történt. |
| c_hash |Kód kivonatértéket. Ez a jogkivonat tartalma kivonatát. |
| Exp |[Lejárati idő]. Az idő, amely után a jogkivonatot már nem fogadható el. |
| IAT |Kiállított. Az az időpont, amikor a jogkivonat lett kiállítva. |
| iss |Kibocsátó. Ez a jogcím értéke mindig az erőforráspartner az AD FS. |
| név |A felhasználó neve. Például: `john@corp.fabrikam.com` |
| nameidentifier |[Névazonosító]. A neve, amelyhez a jogkivonatot adta ki az entitás azonosítója. |
| egyszeri |Munkamenet egyszeri. Az AD FS-ismétléses támadások megelőzése érdekében által generált egyedi érték. |
| egyszerű felhasználónév |Egyszerű felhasználónév (UPN). Például: `john@corp.fabrikam.com` |
| pwd_exp |Jelszó lejárati idejét. Amíg a felhasználó jelszavát vagy egy hasonló hitelesítés titkos, például a PIN-kód másodpercek számát. lejár. |

> [!NOTE]
> A "iss" jogcím tartalmazza az AD FS, a partner (általában ez a jogcím azonosítja a SaaS-szolgáltató a kibocsátó). Azt határozza meg az ügyfél AD FS. Az egyszerű felhasználónév részeként az ügyfél tartományban található.

Ez a cikk ismerteti, hogyan lehet az RP-ből (az alkalmazás) és a fiókpartner (az ügyfél) közötti bizalmi kapcsolat beállítása.

## <a name="ad-fs-deployment"></a>Az AD FS üzembe helyezése

Az SaaS-szolgáltató telepítheti az AD FS helyszíni vagy Azure virtuális gépeken. A biztonsági és rendelkezésre állás a következő irányelveket fontosak:

* Legalább két AD FS-kiszolgálók és a két az AD FS szolgáltatás ajánlott rendelkezésre állásának eléréséhez az AD FS-proxykiszolgáló üzembe helyezése.
* A tartományvezérlők és AD FS-kiszolgálók kell soha nem szabad közvetlenül elérhetővé tenni az interneten, és a közvetlen hozzáférést a azokat egy virtuális hálózatban kell lennie.
* Webalkalmazás-proxyt (korábban az AD FS proxy) közzététele AD FS-kiszolgálók internetkapcsolattal kell használni.

Állítsa be a hasonló topológia az Azure virtuális hálózatok, NSG, az azure virtuális gép és a rendelkezésre állási csoportok használatát igényli. További részletekért lásd: [központi telepítése Windows Server Active Directory az Azure Virtual Machinesben irányelvek][active-directory-on-azure].

## <a name="configure-openid-connect-authentication-with-ad-fs"></a>OpenID Connect-hitelesítés konfigurálása az AD FS-sel

Az SaaS-szolgáltató engedélyeznie kell a OpenID Connect az alkalmazások és az Active Directory összevonási szolgáltatások között. Ehhez adja hozzá az AD FS-ben egy csoportot.  Ezen részletes utasítások találhatók [blogbejegyzés], a "Webes alkalmazás beállítása az OpenId Connect jelentkezzen be az AD FS."

Ezután konfigurálja az OpenID Connect közbenső szoftvert. A metaadatok végpontja `https://domain/adfs/.well-known/openid-configuration`, ahol a tartomány az SaaS-szolgáltatónak az AD FS-tartomány.

Általában akkor lehet, hogy kombinálva ez más OpenID Connect végpontok (például az AAD-). Két különböző bejelentkezési gombot vagy más módon megkülönböztetésükhöz, úgy, hogy a felhasználó a megfelelő hitelesítési végpont küldendő kell.

## <a name="configure-the-ad-fs-resource-partner"></a>Az AD FS erőforráspartner konfigurálása

Az SaaS-szolgáltató minden egyes ügyfél, amely szeretne az ADFS-n keresztül csatlakozik a következőket kell tennie:

1. Jogcímszolgáltatói megbízhatóság hozzáadása.
2. Adja hozzá a jogcímszabályokat.
3. A kezdőtartomány felderítés engedélyezéséhez.

További részleteket az alábbiakban a lépéseket.

### <a name="add-the-claims-provider-trust"></a>A jogcím-szolgáltatói megbízhatóság hozzáadása

1. A Kiszolgálókezelőben kattintson a **eszközök**, majd válassza ki **AD FS-kezelőben**.
2. A konzolfán a **az AD FS**, kattintson a jobb gombbal **jogcím-szolgáltatói Megbízhatóságok**. Válassza ki **jogcím-szolgáltatói megbízhatóság hozzáadása**.
3. Kattintson a **Start** varázsló elindításához.
4. Válassza ki az beállítás "importálási adatok a jogcímszolgáltató, illetve online vagy helyi hálózaton közzétett kapcsolatban". Adja meg az ügyfél összevonási metaadatok végpontjának URI. (Példa: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`.) Szüksége lesz a probléma az ügyféltől.
5. Fejezze be a varázslót, az alapértelmezett beállításokat használva.

### <a name="edit-claims-rules"></a>Jogcímszabályok szerkesztése

1. Kattintson a jobb gombbal az újonnan hozzáadott jogcím-szolgáltatói megbízhatóság, és válassza ki **Jogcímszabályok szerkesztése**.
2. Kattintson a **szabály hozzáadása**.
3. Válassza ki a "Haladnak keresztül vagy a szűrő egy bejövő jogcím", és kattintson a **tovább**.
   ![Átalakítási jogcímszabály hozzáadása varázsló](./images/edit-claims-rule.png)
4. Adja meg a szabály nevét.
5. Válassza a "Bejövő jogcím típusa" **UPN**.
6. Válassza ki a "Az összes jogcímérték továbbítása".
   ![Átalakítási jogcímszabály hozzáadása varázsló](./images/edit-claims-rule2.png)
7. Kattintson a **Befejezés** gombra.
8. Ismételje meg a 2 – 7, és adja meg **Forráshorgony jogcím típusa** számára a bejövő jogcím típusa.
9. Kattintson a **OK** a varázsló befejezéséhez.

### <a name="enable-home-realm-discovery"></a>A kezdőtartomány felderítés engedélyezéséhez

Futtassa a következő PowerShell-parancsfájlt:

```powershell
Set-ADFSClaimsProviderTrust -TargetName "name" -OrganizationalAccountSuffix @("suffix")
```

Ha "name" a valódi neve az a jogcím-szolgáltatói megbízhatóság, és "utótag" az ügyfél az UPN-utótagot a AD, (például "corp.fabrikam.com").

Ezzel a konfigurációval a végfelhasználók írja be a saját szervezeti fiókjukba, és az AD FS automatikusan kiválasztja a megfelelő jogcímszolgáltatót. Lásd: [az AD FS bejelentkezési oldalainak testreszabása], "Configure identitásszolgáltató bizonyos e-mail utótagok használata" területen.

## <a name="configure-the-ad-fs-account-partner"></a>Az AD FS-fiók partnerének beállítása

Az ügyfél a következőket kell tennie:

1. Adjon hozzá egy függő entitásonkénti (RP) megbízhatóságát.
2. Hozzáadja a jogcímszabályokat.

### <a name="add-the-rp-trust"></a>A függő Entitás megbízhatóságának hozzáadása

1. A Kiszolgálókezelőben kattintson a **eszközök**, majd válassza ki **AD FS-kezelőben**.
2. A konzolfán a **az AD FS**, kattintson a jobb gombbal **függő entitás Megbízhatóságai**. Válassza ki **függő entitás megbízhatóságának hozzáadása**.
3. Válassza ki **jogcímeket figyelembe** kattintson **Start**.
4. Az a **adatforrás kiválasztása** lapra, jelölje be a "Importálási adatok a jogcímszolgáltató közzé online vagy helyi hálózaton" lehetőséget. Adja meg a SaaS-szolgáltató összevonási metaadatok végpontjának URI.
   ![Adja hozzá a függő entitás megbízhatóságának varázsló](./images/add-rp-trust.png)
5. Az a **megjelenítendő név megadása** lapon, bármilyen nevet adja.
6. Az a **válassza a hozzáférés-vezérlési házirend** lapon, válassza ki a szabályzatot. A szervezet engedélyezése mindenki számára, vagy egy adott biztonsági csoport kiválasztása.
   ![Adja hozzá a függő entitás megbízhatóságának varázsló](./images/add-rp-trust2.png)
7. Adja meg a szükséges paramétereket a **házirend** mezőbe.
8. Kattintson a **tovább** a varázsló befejezéséhez.

### <a name="add-claims-rules"></a>A szabályok hozzáadása

1. Kattintson a jobb gombbal az újonnan hozzáadott függőentitás-megbízhatóságot, és válassza ki **jogcím-kiállítási szabályzat szerkesztése**.
2. Kattintson a **szabály hozzáadása**.
3. Válassza ki a "Küldés LDAP attribútumok szerint jogcímek", és kattintson a **tovább**.
4. Adjon meg egy nevet a szabálynak, például az "Egyszerű".
5. A **attribútumtár**válassza **Active Directory**.
   ![Átalakítási jogcímszabály hozzáadása varázsló](./images/add-claims-rules.png)
6. Az a **LDAP attribútumok leképezése** szakaszban:
   * A **LDAP attribútum**válassza **felhasználónév-egyszerű**.
   * A **kimenő jogcímtípus**válassza **UPN**.
     ![Átalakítási jogcímszabály hozzáadása varázsló](./images/add-claims-rules2.png)
7. Kattintson a **Befejezés** gombra.
8. Kattintson a **szabály hozzáadása** újra.
9. Válassza ki a "Küldés jogcímek használata egy egyéni szabály", és kattintson a **tovább**.
10. Adjon meg egy nevet a szabálynak, például a "Forráshorgony jogcímtípus".
11. A **egyéni szabály**, írja be a következőket:

    ```console
    EXISTS([Type == "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype"])=>
    issue (Type = "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype",
          Value = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn");
    ```

    Ez a szabály kiad egy jogcímet típusú `anchorclaimtype`. A jogcím arra utasítja a függő entitás használandó egyszerű felhasználónév a felhasználó azonosítója nem módosítható.
12. Kattintson a **Befejezés** gombra.
13. Kattintson a **OK** a varázsló befejezéséhez.

<!-- links -->

[Azure AD Connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
[összevonási megbízhatósági kapcsolatot]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[fiókpartner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[erőforráspartner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[Azonnali hitelesítés]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[Lejárati idő]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[Névazonosító]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[active-directory-on-azure]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[Blogbejegyzés]: https://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[Az AD FS bejelentkezési oldalainak testreszabása]: https://technet.microsoft.com/library/dn280950.aspx
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[client assertion]: client-assertion.md
[active-directory-dotnet-webapp-wsfederation]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation
