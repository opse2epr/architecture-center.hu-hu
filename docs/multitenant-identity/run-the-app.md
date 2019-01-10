---
title: A Surveys alkalmazás futtatása
description: Hogyan lehet a Surveys mintaalkalmazás helyi futtatásához.
author: MikeWasson
ms.date: 07/21/2017
ms.openlocfilehash: b73eeb04755b3dc8443b215bb034c82e3095681d
ms.sourcegitcommit: 7d9efe716e8c9e99f3fafa9d0213d48c23d9713d
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/09/2019
ms.locfileid: "54160757"
---
# <a name="run-the-surveys-application"></a>A Surveys alkalmazás futtatása

Ez a cikk azt ismerteti, hogyan futtathat a [Tailspin Surveys](./tailspin.md) alkalmazást helyileg, a Visual Studióból. Ezeket a lépéseket az Azure-ban az alkalmazás nem telepít. Azonban szüksége lesz egy Azure-erőforrások létrehozása &mdash; egy Azure Active Directory (Azure AD) címtár és a egy Redis gyorsítótárhoz.

A következő lépések összefoglalása:

1. A Tailspin fiktív hozzon létre egy Azure AD-címtár (bérlő).
2. A Surveys alkalmazás és a háttéralkalmazás webes API regisztrálása az Azure ad-ben.
3. Hozzon létre egy Azure Redis Cache-példányhoz.
4. Az Alkalmazásbeállítások konfigurálása, és hozzon létre egy helyi adatbázist.
5. Futtassa az alkalmazást, és regisztráljon egy új bérlőt.
6. Alkalmazás-szerepkörök hozzáadása a felhasználók számára.

## <a name="prerequisites"></a>Előfeltételek

- [A Visual Studio 2017] [ VS2017] együtt a [ASP.NET- és fejlesztési számítási feladatot](https://visualstudio.microsoft.com/vs/support/selecting-workloads-visual-studio-2017) telepítve
- [A Microsoft Azure](https://azure.microsoft.com) fiók

## <a name="create-the-tailspin-tenant"></a>A Tailspin bérlő létrehozása

Tailspin a fiktív vállalat, amely futtatja a Surveys alkalmazás. Tailspin regisztrálni az alkalmazást a többi bérlő engedélyezése az Azure AD. Ezek az ügyfelek ezután használhatja az Azure AD hitelesítő adataikkal bejelentkeznie az alkalmazásba.

Ebben a lépésben az Azure AD-címtárat a Tailspin fog létrehozni.

1. Jelentkezzen be az [Azure Portalra][portal].

2. Kattintson a **+ erőforrás létrehozása** > **identitás** > **az Azure Active Directory**.

3. Adja meg `Tailspin` a szervezet neve, és adjon meg tartománynevet. A tartomány neve lesz a képernyő `xxxx.onmicrosoft.com` és globálisan egyedinek kell lennie.

    ![Könyvtár párbeszédpanel létrehozása](./images/running-the-app/new-tenant.png)

4. Kattintson a **Create** (Létrehozás) gombra. Az új könyvtár létrehozása néhány percet igénybe vehet.

A végpontok közötti forgatókönyv végrehajtásához szüksége lesz egy második Azure AD-címtár, amely jelöli, amelyet az alkalmazás regisztrál. Használja az alapértelmezett Azure AD-címtár (nem a Tailspin), vagy hozzon létre egy új könyvtárat erre a célra. A példákban a Contoso a fiktív ügyfélként használjuk.

## <a name="register-the-surveys-web-api"></a>A felmérések webes API regisztrálása

1. Az a [az Azure portal][portal], váltson az új Tailspin könyvtárat a portál jobb felső sarkában a fiók kiválasztásával.

2. A bal oldali navigációs ablaktáblán válassza ki a **Azure Active Directory**.

3. Kattintson a **alkalmazásregisztrációk** > **új alkalmazásregisztráció**.

4. Az a **létrehozás** panelen adja meg a következőket:

   - **Név**: `Surveys.WebAPI`

   - **Az alkalmazástípus**: `Web app / API`

   - **Bejelentkezés URL-cím**: `https://localhost:44301/`

   ![Képernyőfelvétel a webes API regisztrálása](./images/running-the-app/register-web-api.png)

5. Kattintson a **Create** (Létrehozás) gombra.

6. Az a **alkalmazásregisztrációk** panelen válassza ki az új **Surveys.WebAPI** alkalmazás.

7. Kattintson a **beállítások** > **tulajdonságok**.

8. Az a **Alkalmazásazonosító URI-t** beviteli mező, adja meg `https://<domain>/surveys.webapi`, ahol `<domain>` tartományneve annak a címtárnak. Például:`https://tailspin.onmicrosoft.com/surveys.webapi`

    ![Beállítások](./images/running-the-app/settings.png)

9. Állítsa be **több-bérlős** való **Igen**.

10. Kattintson a **Save** (Mentés) gombra.

## <a name="register-the-surveys-web-app"></a>A felmérések webes alkalmazás regisztrálása

1. Lépjen vissza a **alkalmazásregisztrációk** panelen, és kattintson **új alkalmazásregisztráció**.

2. Az a **létrehozás** panelen adja meg a következőket:

    - **Név**: `Surveys`
    - **Az alkalmazástípus**: `Web app / API`
    - **Bejelentkezés URL-cím**: `https://localhost:44300/`

    Figyelje meg, hogy a bejelentkezési URL-rendelkezik-e a port számát a `Surveys.WebAPI` alkalmazás az előző lépésben.

3. Kattintson a **Create** (Létrehozás) gombra.

4. Az a **alkalmazásregisztrációk** panelen válassza ki az új **felmérések** alkalmazás.

5. Másolja az alkalmazás azonosítója. Ez később lesz szüksége.

    ![Képernyőkép az Alkalmazásazonosító másolása](./images/running-the-app/application-id.png)

6. Kattintson a **Tulajdonságok** elemre.

7. Az a **Alkalmazásazonosító URI-t** beviteli mező, adja meg `https://<domain>/surveys`, ahol `<domain>` tartományneve annak a címtárnak.

    ![Beállítások](./images/running-the-app/settings.png)

8. Állítsa be **több-bérlős** való **Igen**.

9. Kattintson a **Save** (Mentés) gombra.

10. Az a **beállítások** panelen kattintson a **válasz URL-címek**.

11. Adja hozzá a következő válasz URL-cím: `https://localhost:44300/signin-oidc`.

12. Kattintson a **Save** (Mentés) gombra.

13. A **API-hozzáférés**, kattintson a **kulcsok**.

14. Adja meg a leírását, például `client secret`.

15. Az a **kiválasztása időtartama** legördülő menüben válasszon ki **1 év**.

16. Kattintson a **Save** (Mentés) gombra. A kulcs jön létre, amikor menti.

17. Mielőtt ezt a panelt elhagyni, másolja a kulcs értékét.

    > [!NOTE]
    > A kulcs nem lesz látható újra után meg elhagyni a panelt.

18. A **API-hozzáférés**, kattintson a **szükséges engedélyek**.

19. Kattintson a **hozzáadása** > **API kiválasztása**.

20. A keresőmezőbe keresése `Surveys.WebAPI`.

    ![Engedélyek](./images/running-the-app/permissions.png)

21. Válassza ki `Surveys.WebAPI` kattintson **kiválasztása**.

22. A **delegált engedélyek**, ellenőrizze **hozzáférés Surveys.WebAPI**.

    ![Engedélyek beállítása meghatalmazott](./images/running-the-app/delegated-permissions.png)

23. Kattintson a **Kiválasztás** > **Kész** gombra.

## <a name="update-the-application-manifests"></a>Frissítse az alkalmazásjegyzékeket

1. Lépjen vissza a **beállítások** paneljén a `Surveys.WebAPI` alkalmazást.

2. Kattintson a **Manifest** > **szerkesztése**.

    ![Képernyőkép az alkalmazásjegyzék szerkesztése](./images/running-the-app/manifest.png)

3. Adja hozzá a következő JSON-a `appRoles` elemet. Hozzon létre új GUID-azonosítói a `id` tulajdonságait.

   ```json
   {
     "allowedMemberTypes": ["User"],
     "description": "Creators can create surveys",
     "displayName": "SurveyCreator",
     "id": "<Generate a new GUID. Example: 1b4f816e-5eaf-48b9-8613-7923830595ad>",
     "isEnabled": true,
     "value": "SurveyCreator"
   },
   {
     "allowedMemberTypes": ["User"],
     "description": "Administrators can manage the surveys in their tenant",
     "displayName": "SurveyAdmin",
     "id": "<Generate a new GUID>",  
     "isEnabled": true,
     "value": "SurveyAdmin"
   }
   ```

4. Az a `knownClientApplications` tulajdonság, adja hozzá a felmérések webes alkalmazás, amely korábban a Surveys alkalmazás regisztrálásakor kapott Alkalmazásazonosító. Példa:

   ```json
   "knownClientApplications": ["be2cea23-aa0e-4e98-8b21-2963d494912e"],
   ```

   Ez a beállítás a Surveys alkalmazás hozzáadása a webes API hívása jogosult ügyfelek listáját.

5. Kattintson a **Save** (Mentés) gombra.

Most ismételje meg a Surveys alkalmazás ugyanezekkel a lépésekkel kivéve ne adjon hozzá egy bejegyzés `knownClientApplications`. Használja ugyanazt a szerepkör-definíciók, de új GUID azonosítóját hozza létre.

## <a name="create-a-new-redis-cache-instance"></a>Új Redis Cache-példány létrehozása

A Surveys alkalmazás a Redis cache OAuth 2 hozzáférési jogkivonatok használja. A gyorsítótár létrehozásához:

1. Lépjen a [az Azure Portal](https://portal.azure.com) kattintson **+ erőforrás létrehozása** > **adatbázisok** > **Redis Cache**.

2. Adja meg a szükséges információkat, beleértve a DNS-név, erőforráscsoport, helyét, és a tarifacsomag. Hozzon létre egy új erőforráscsoportot, vagy használjon egy meglévő erőforráscsoportot.

3. Kattintson a **Create** (Létrehozás) gombra.

4. A Redis gyorsítótár létrehozása után keresse meg az erőforrást a portálon.

5. Kattintson a **hozzáférési kulcsok** , és másolja az elsődleges kulcsot.

Egy Redis cache létrehozásával kapcsolatos további információkért lásd: [How to Azure Redis Cache használata](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache).

## <a name="set-application-secrets"></a>Titkos alkalmazáskulcsok beállítása

1. Nyissa meg a Tailspin.Surveys megoldást a Visual Studióban.

2. A Megoldáskezelőben kattintson a jobb gombbal a Tailspin.Surveys.Web projektet, és válassza ki **felhasználói titkok kezelése**.

3. A secrets.json fájlban illessze be a következőket:

    ```json
    {
      "AzureAd": {
        "ClientId": "<Surveys application ID>",
        "ClientSecret": "<Surveys app client secret>",
        "PostLogoutRedirectUri": "https://localhost:44300/",
        "WebApiResourceId": "<Surveys.WebAPI app ID URI>"
      },
      "Redis": {
        "Configuration": "<Redis DNS name>.redis.cache.windows.net,password=<Redis primary key>,ssl=true"
      }
    }
    ```

    Cserélje le a csúcsos zárójelpárban van, a következő megjelenített elemek:

    - `AzureAd:ClientId`: Az a Surveys alkalmazás azonosítója.
    - `AzureAd:ClientSecret`: Az Azure ad-ben a Surveys alkalmazás regisztrációja során létrehozott kulcs.
    - `AzureAd:WebApiResourceId`: Az Alkalmazásazonosító URI-t, hogy a megadott Azure AD-ben a Surveys.WebAPI alkalmazás létrehozásakor. Az űrlap kell rendelkeznie `https://<directory>.onmicrosoft.com/surveys.webapi`
    - `Redis:Configuration`: Ez a karakterlánc, a DNS-nevét a Redis cache és az elsődleges elérési kulcs létrehozása. For example, "tailspin.redis.cache.windows.net,password=2h5tBxxx,ssl=true".

4. Mentse a frissített secrets.json fájlt.

5. Ismételje meg ezeket a lépéseket a Tailspin.Surveys.WebAPI projekt, de secrets.json illessze be a következő. Ahogy korábban is cserélje le a csúcsos zárójelpárban van, a cikkeket.

    ```json
    {
      "AzureAd": {
        "WebApiResourceId": "<Surveys.WebAPI app ID URI>"
      },
      "Redis": {
        "Configuration": "<Redis DNS name>.redis.cache.windows.net,password=<Redis primary key>,ssl=true"
      }
    }
    ```

## <a name="initialize-the-database"></a>Az adatbázis inicializálása

Ebben a lépésben használandó entitás-keretrendszer 7 LocalDB használatakor egy helyi SQL-adatbázis létrehozásához.

1. Nyisson meg egy parancsablakot

2. Keresse meg a Tailspin.Surveys.Data projekt.

3. Futtassa az alábbi parancsot:

    ```bat
    dotnet ef database update --startup-project ..\Tailspin.Surveys.Web
    ```

## <a name="run-the-application"></a>Az alkalmazás futtatása

Futtassa az alkalmazást, indítsa el a Tailspin.Surveys.Web és a Tailspin.Surveys.WebAPI projektek.

Visual Studio segítségével mindkét projekt futtatásához automatikusan F5 billentyűt, a következőképpen állíthatja be:

1. A Megoldáskezelőben kattintson a jobb gombbal a megoldás, és kattintson a **indítási projektek beállítása**.
2. Válassza ki **több kezdőprojekt**.
3. Állítsa be **művelet** = **Start** Tailspin.Surveys.Web és Tailspin.Surveys.WebAPI projektek.

## <a name="sign-up-a-new-tenant"></a>Regisztráljon egy új bérlőt

Az alkalmazás indításakor nem jelentkezett be, így az üdvözlő oldal jelenik meg:

![Kezdőlap](./images/running-the-app/screenshot1.png)

Regisztráció a szervezet:

1. Kattintson a **regisztrálni a vállalat a Tailspin**.
2. Jelentkezzen be az Azure AD-címtár, amely a szervezet a Surveys alkalmazással jelöli. Rendszergazda felhasználóként jelentkezzen be.
3. Fogadja el a beleegyezést kérő üzenetet.

Az alkalmazás a bérlő regisztrálja, és ezután jelentkezteti ki. Az alkalmazás kijelentkezik, mert szüksége lesz az alkalmazás-szerepkörök beállítása az Azure AD-ben az alkalmazás használata előtt.

![Regisztráció után](./images/running-the-app/screenshot2.png)

## <a name="assign-application-roles"></a>Alkalmazás-szerepkörök hozzárendelése

Amikor egy bérlő regisztrál, a bérlői rendszergazda AD hozzá kell rendelnie alkalmazás-szerepkörök a felhasználók számára.

1. Az a [az Azure portal][portal], váltson át az Azure AD-címtár, amellyel a Surveys alkalmazás regisztrálhat.

2. A bal oldali navigációs ablaktáblán válassza ki a **Azure Active Directory**.

3. Kattintson a **vállalati alkalmazások** > **minden alkalmazás**. A portálon megjelennek `Survey` és `Survey.WebAPI`. Ha nem, akkor győződjön meg arról, hogy elvégezte-e a regisztrációs folyamat.

4. Kattintson a Surveys alkalmazás.

5. Kattintson a **felhasználók és csoportok**.

6. Kattintson a **Felhasználó hozzáadása** parancsra.

7. Ha rendelkezik Azure AD Premium szolgáltatással, kattintson a **felhasználók és csoportok**. Ellenkező esetben kattintson a **felhasználók**. (A prémium szintű Azure AD szerepkör hozzárendelése egy csoporthoz szükséges.)

8. Válassza ki egy vagy több felhasználót, és kattintson a **kiválasztása**.

    ![Felhasználó vagy csoport kiválasztása](./images/running-the-app/select-user-or-group.png)

9. Válassza ki a szerepkört, és kattintson a **kiválasztása**.

    ![Felhasználó vagy csoport kiválasztása](./images/running-the-app/select-role.png)

10. Kattintson a **Hozzárendelés** gombra.

Ismételje meg a ugyanazokat a lépéseket, ha Survey.WebAPI alkalmazásához szerepköröket.

> [!IMPORTANT]
> A felhasználó a felmérési és a Survey.WebAPI ugyanezeket a szerepköröket mindig rendelkeznie kell. Ellenkező esetben a felhasználói jogosultak lesznek inkonzisztens, amely a webes API a 403-as (tiltott) hibákhoz vezethetnek.

Most lépjen vissza az alkalmazást, és jelentkezzen be újra. Kattintson a **saját felmérések**. Ha a felhasználói szerepkör van rendelve, a SurveyAdmin vagy SurveyCreator, látni fogja a **felmérés létrehozása** gomb, amely azt jelzi, hogy a felhasználók számára hozzon létre egy új felmérésre.

![A felmérések](./images/running-the-app/screenshot3.png)

<!-- links -->

[portal]: https://portal.azure.com
[VS2017]: https://www.visualstudio.com/vs/
