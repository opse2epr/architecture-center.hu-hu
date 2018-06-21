---
title: Összevonás az ügyfél AD FS szolgáltatásával
description: Hogyan való federate az ügyfél által az AD FS egy több-bérlős alkalmazásban
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: token-cache
pnp.series.next: client-assertion
ms.openlocfilehash: 08bf567085a940287de310f61b9f447d0ce5d5ec
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/23/2018
ms.locfileid: "29477444"
---
# <a name="federate-with-a-customers-ad-fs"></a>Összevonás az ügyfél AD FS szolgáltatásával

Ez a cikk ismerteti, hogyan egy több-bérlős SaaS-alkalmazáshoz is támogatják a hitelesítés az Active Directory összevonási szolgáltatások (AD FS) keresztül ahhoz, hogy az ügyfél az AD FS összevonni.

## <a name="overview"></a>Áttekintés
Azure Active Directory (Azure AD) megkönnyíti, hogy a bejelentkezéshez a felhasználók az Azure AD bérlőre, beleértve az Office365 és a Dynamics CRM Online használó ügyfelek számára. De mi a helyzet felhasználókat a helyszíni Active Directory a vállalati intraneten?

Egy elem ezen ügyfelek szinkronizálása a helyszíni AD az Azure AD használatával [az Azure AD Connect]. Egyes felhasználók azonban nem tudja használni ezt a módszert, a vállalati informatikai házirend miatt, vagy más okból lehet. Ebben az esetben egy másik lehetőség egy Active Directory összevonási szolgáltatások (AD FS) keresztül összevonni.

Ebben a forgatókönyvben engedélyezése:

* Az ügyfél internetre AD FS farmot kell rendelkeznie.
* A Szolgáltatottszoftver-szolgáltatót telepíti a saját AD FS-farm.
* Az ügyfél és a Szolgáltatottszoftver-szolgáltató be kell állítania [összevonási megbízhatósági kapcsolat]. Ez az manuális folyamat.

A megbízhatósági kapcsolat három fő szerepkör is létezik:

* Az ügyfél Active Directory összevonási szolgáltatások a [fiókpartner]felelős a felhasználók az ügyfelek hitelesítéséhez, AD, és biztonsági jogkivonatokat hoz létre felhasználói jogcímeket.
* A Szolgáltatottszoftver-szolgáltató AD FS a [erőforráspartner], amely a fiókpartner megbízik, és megkapja a felhasználói jogcímeket.
* Az alkalmazás egy függő entitásonkénti (RP) a Szolgáltatottszoftver-szolgáltató AD FS-ként van konfigurálva.
  
  ![Összevonási megbízhatósági kapcsolat](./images/federation-trust.png)

> [!NOTE]
> Ez a cikk azt feltételezzük, hogy az alkalmazás által használt OpenID connect hitelesítési protokoll. Egy másik lehetőség, hogy használja a WS-Federation.
> 
> A Szolgáltatottszoftver-szolgáltató OpenID Connect, az AD FS 2016 rendszert futtató Windows Server 2016 kell használnia. Az AD FS 3.0 nem támogatja az OpenID Connect.
> 
> Az ASP.NET Core nem tartalmazza a WS-Federation out-of-az-box támogatása.
> 
> 

Például egy ASP.NET 4 WS-Federation használ, tekintse meg a [active-directory-dotnet-webalkalmazás-wsfederation minta][active-directory-dotnet-webapp-wsfederation].

## <a name="authentication-flow"></a>Hitelesítési folyamat
1. Amikor a felhasználó a "bejelentkezés" kattint, az alkalmazás átirányítja a Szolgáltatottszoftver-szolgáltató az AD FS OpenID Connect végpontja.
2. A felhasználó megadja a szervezeti felhasználói nevét ("`alice@corp.contoso.com`"). Az AD FS átirányítja az ügyfél Active Directory összevonási szolgáltatások, ahol a felhasználó megadja a hitelesítő adatokat használja a hitelesítőtartomány feltárását.
3. Az ügyfél az AD FS küldi el a Szolgáltatottszoftver-szolgáltató AD FS-re, felhasználói jogcímeket WF-összevonás (vagy SAML) használatával.
4. Jogcímek az AD FS áramolnak az alkalmazásba, az OpenID Connect használatával. Ehhez a WS-Federation protokoll átmenetet.

## <a name="limitations"></a>Korlátozások
Alapértelmezés szerint a függő entitás alkalmazás csak meghatározott érhető el a id_token, az alábbi táblázatban szereplő jogcímeket kap. Az AD FS 2016 testre szabhatja a id_token OpenID Connect forgatókönyvekben. További információkért lásd: [az AD FS egyéni azonosító-jogkivonatokat](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016).

| Jogcím | Leírás |
| --- | --- |
| és |A célközönség. Az alkalmazás, amelyhez a jogcímek kiállítása volt. |
| AuthenticationInstant |[Hitelesítési azonnali]. Az időpontot, mely hitelesítési történt. |
| c_hash |Kód kivonat értéke. Ez a token tartalmának kivonatát. |
| Exp |[Lejárati idő]. Az az idő, amely után a jogkivonatot már nem fogja elfogadni. |
| IAT |Kiállított. Az az idő, amikor a jogkivonat lett kiállítva. |
| iss |Kibocsátó. A jogcím értéke mindig az erőforráspartner AD FS. |
| név |A felhasználónév. Példa: `john@corp.fabrikam.com` |
| NameIdentifier |[Névazonosítója]. A neve, amelynek a token ki az entitás azonosítója. |
| Nonce |Munkamenet nonce. Ismétléses támadások megelőzése érdekében az AD FS által létrehozott egyedi értéket. |
| egyszerű felhasználónév |Egyszerű felhasználónév (UPN). Példa: `john@corp.fabrikam.com` |
| pwd_exp |Jelszó lejárati időt. Mindaddig, amíg a felhasználó jelszava vagy hasonló hitelesítési titkos kulcs, például a PIN-kód másodpercek számát. lejár. |

> [!NOTE]
> A "iss" jogcímek az AD FS, a partner tartalmazza (általában a jogcím azonosítja a Szolgáltatottszoftver-szolgáltató a kiállítóként). Azt határozza meg az ügyfél Active Directory összevonási szolgáltatások. A felhasználónév részeként a felhasználói tartományban található.
> 
> 

Ez a cikk többi ismerteti a megbízhatósági kapcsolat, a függő Entitás (alkalmazás) és a fiókpartner (az ügyfél) között.

## <a name="ad-fs-deployment"></a>Az AD FS üzembe helyezése
A Szolgáltatottszoftver-szolgáltatót telepítheti az AD FS helyszíni vagy Azure virtuális gépeken. A biztonsági és rendelkezésre állás a következő irányelveket fontosak:

* Legalább két AD FS-kiszolgálót telepíteni, két AD FS-proxy kiszolgáló ajánlott az AD FS szolgáltatás rendelkezésre állásának eléréséhez.
* Tartományvezérlők és AD FS-kiszolgáló kell soha nem szabad közvetlenül elérhetővé tenni az interneten, és azokat közvetlen hozzáféréssel rendelkező virtuális hálózatban kell lennie.
* Webalkalmazás-proxyt (korábban az AD FS proxy) kell használni az AD FS-kiszolgáló közzététele az interneten.

Beállítása az Azure-ban egy hasonló topológia virtuális hálózatok, a NSG meg, az azure virtuális gép és a rendelkezésre állási készletek használatát igényli. További részletekért lásd: [telepítése Windows Server Active Directory telepítése Azure virtuális gépekre vonatkozó irányelvek][active-directory-on-azure].

## <a name="configure-openid-connect-authentication-with-ad-fs"></a>Az AD FS OpenID Connect hitelesítési konfigurálása
A Szolgáltatottszoftver-szolgáltató engedélyeznie kell a OpenID Connect az alkalmazások és az Active Directory összevonási szolgáltatások között. Ehhez adja hozzá az AD FS alkalmazáscsoport.  Részletes utasításokat talál ezen [blogbejegyzés], a "Az OpenId Connect egy webes alkalmazás beállítása az AD FS bejelentkezni." 

Ezután konfigurálja az OpenID Connect köztes. A metaadat-végpont esetében `https://domain/adfs/.well-known/openid-configuration`, ahol a tartomány a Szolgáltatottszoftver-szolgáltató az AD FS-tartomány.

Általában akkor előfordulhat, hogy egyesíthető ez más OpenID Connect végpontok (például az aad-ben). Szüksége lesz két különböző bejelentkezési gomb vagy megkülönböztetésükhöz, más módon, hogy a felhasználó a rendszer elküldi a megfelelő hitelesítési végpontot.

## <a name="configure-the-ad-fs-resource-partner"></a>Az AD FS erőforráspartner konfigurálása
A Szolgáltatottszoftver-szolgáltató, amelyet csatlakoztatni AD FS révén minden ügyfél esetében a következőket kell tennie:

1. Jogcímszolgáltatói megbízhatóság hozzáadása.
2. Adja hozzá a jogcímszabályokat.
3. A kezdőtartomány felderítés engedélyezéséhez.

A lépések a következők részletesebben.

### <a name="add-the-claims-provider-trust"></a>A jogcím-szolgáltatói megbízhatóság hozzáadása
1. A Kiszolgálókezelőben kattintson **eszközök**, majd válassza ki **AD FS felügyeleti**.
2. A konzolfán a **AD FS**, kattintson a jobb gombbal **jogcím-szolgáltatói Megbízhatóságok**. Válassza ki **jogcím-szolgáltatói megbízhatóság hozzáadása**.
3. Kattintson a **Start** varázsló elindításához.
4. Válassza ki a beállítást "Import adatait a jogcím-szolgáltató online vagy helyi hálózaton közzétett". Adja meg az ügyfél összevonási metaadatok végpont URI. (Példa: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`.) Szüksége lesz az ügyfél kiindulási.
5. A varázsló az alapértelmezett beállításokat használva.

### <a name="edit-claims-rules"></a>Jogcímszabályok szerkesztése
1. Kattintson a jobb gombbal az újonnan hozzáadott jogcím-szolgáltatói megbízhatóság, és válassza ki **Jogcímszabályok szerkesztése**.
2. Kattintson a **szabály hozzáadása**.
3. Válassza ki a "Átadni keresztül vagy szűrő egy bejövő jogcím", és kattintson a **következő**.
   ![Átalakítási jogcímszabály hozzáadása varázsló](./images/edit-claims-rule.png)
4. Adja meg a szabály nevét.
5. Válassza ki a "Bejövő jogcím típusa" **UPN**.
6. Válassza ki a "Az összes jogcímérték továbbítása".
   ![Átalakítási jogcímszabály hozzáadása varázsló](./images/edit-claims-rule2.png)
7. Kattintson a **Befejezés** gombra.
8. Ismételje meg a 2 – 7, és adja meg a **horgonyzási jogcím típusa** a bejövő jogcím típusa.
9. Kattintson a **OK** a varázsló befejezéséhez.

### <a name="enable-home-realm-discovery"></a>A kezdőtartomány felderítés engedélyezéséhez
Futtassa a következő PowerShell-parancsfájlt:

```
Set-ADFSClaimsProviderTrust -TargetName "name" -OrganizationalAccountSuffix @("suffix")
```

ahol "name" a jogcím-szolgáltatói megbízhatóság rövid nevét, és "utótag" az egyszerű Felhasználónévi utótagot az ügyfél által AD (például "corp.fabrikam.com").

Ezzel a konfigurációval a végfelhasználók maguk írhassanak bele saját szervezeti fiókjukba, és az AD FS automatikusan kiválasztja a megfelelő jogcímszolgáltatót. Lásd: [az AD FS bejelentkezési oldalainak testreszabása], szakaszban "konfigurálása identitás Provider bizonyos e-mail utótagok használatára".

## <a name="configure-the-ad-fs-account-partner"></a>Az AD FS fiókpartner konfigurálása
Az ügyfél a következőket kell tennie:

1. Adjon hozzá egy megbízható függő entitásonkénti (RP).
2. Hozzáadja a jogcímszabályokat.

### <a name="add-the-rp-trust"></a>A függő Entitás megbízhatóságának hozzáadása
1. A Kiszolgálókezelőben kattintson **eszközök**, majd válassza ki **AD FS felügyeleti**.
2. A konzolfán a **AD FS**, kattintson a jobb gombbal **függő entitás Megbízhatóságai**. Válassza ki **függő entitás megbízhatóságának hozzáadása**.
3. Válassza ki **jogcímeket figyelembe vevő** kattintson **Start**.
4. Az a **adatforrás** lapon, válassza az "Adatok importálása a jogcímszolgáltató közzé online vagy helyi hálózaton" lehetőséget. Írja be a Szolgáltatottszoftver-szolgáltató összevonási metaadatok végpont URI.
   ![Adja hozzá a függő entitás megbízhatóságának varázsló](./images/add-rp-trust.png)
5. Az a **megjelenített név meghatározása** lapján adjon meg egy tetszőleges nevet.
6. Az a **hozzáférés-vezérlési szabályzat kiválasztása** lapon, válassza ki a házirend. A szervezet engedélyezése mindenki számára, vagy válasszon ki egy biztonsági csoportot.
   ![Adja hozzá a függő entitás megbízhatóságának varázsló](./images/add-rp-trust2.png)
7. Adja meg a szükséges paramétereket a **házirend** mezőbe.
8. Kattintson a **tovább** a varázsló befejezéséhez.

### <a name="add-claims-rules"></a>Jogcímszabályok hozzáadása
1. Kattintson a jobb gombbal az újonnan hozzáadott függő entitás megbízhatóságához, és válassza ki **jogcím-kiállítási házirend szerkesztése**.
2. Kattintson a **szabály hozzáadása**.
3. Válassza ki a "Küldési LDAP attribútumok szerint jogcímek", és kattintson a **következő**.
4. Adja meg a szabályhoz, például "Egyszerű" nevét.
5. A **attribútumtár**, jelölje be **Active Directory**.
   ![Átalakítási jogcímszabály hozzáadása varázsló](./images/add-claims-rules.png)
6. Az a **LDAP attribútumok leképezése** szakasz:
   * A **LDAP attribútum**, jelölje be **felhasználónév-egyszerű**.
   * A **kimenő jogcímtípus**, jelölje be **UPN**.
     ![Átalakítási jogcímszabály hozzáadása varázsló](./images/add-claims-rules2.png)
7. Kattintson a **Befejezés** gombra.
8. Kattintson a **szabály hozzáadása** újra.
9. Válassza ki a "Küldési jogcímek használata egy egyéni szabály", és kattintson a **következő**.
10. Adja meg a szabályhoz, például "Horgonyzási jogcímtípus" nevét.
11. A **egyéni szabály**, írja be a következőket:
    
    ```
    EXISTS([Type == "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype"])=>
    issue (Type = "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype",
          Value = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn");
    ```
    
    Ez a szabály akkor ad ki egy jogcímet típusú `anchorclaimtype`. A jogcímek közli a függő entitás egyszerű felhasználónév használata a felhasználói azonosítót nem módosítható.
12. Kattintson a **Befejezés** gombra.
13. Kattintson a **OK** a varázsló befejezéséhez.


<!-- Links -->
[az Azure AD Connect]: /azure/active-directory/active-directory-aadconnect/
[összevonási megbízhatósági kapcsolat]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[fiókpartner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[erőforráspartner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[Hitelesítési azonnali]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[Lejárati idő]: http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[Névazonosítója]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[active-directory-on-azure]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[blogbejegyzés]: http://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[az AD FS bejelentkezési oldalainak testreszabása]: https://technet.microsoft.com/library/dn280950.aspx
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[client assertion]: client-assertion.md
[active-directory-dotnet-webapp-wsfederation]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation
