---
title: A Surveys alkalmazás futtatása
description: A felmérések mintaalkalmazás futtatása helyben
author: MikeWasson
ms:date: 07/21/2017
ms.openlocfilehash: 28d976374e5d6dbad434873eef149704f26a1f3f
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
ms.locfileid: "30848682"
---
# <a name="run-the-surveys-application"></a>A Surveys alkalmazás futtatása

Ez a cikk ismerteti, hogyan futtathatók a [Dejójáték felmérések](./tailspin.md) helyileg, a Visual Studio alkalmazás. Ezeket a lépéseket az Azure alkalmazás nem telepít. Azonban szüksége lesz egy Azure-erőforrások létrehozásához &mdash; az Azure Active Directory (Azure AD) címtár és a Redis gyorsítótár.

A lépések összefoglalása itt található:

1. Hozzon létre egy Azure AD-címtár (bérlői) Dejójáték fiktív cég.
2. Regisztrálja a felmérések alkalmazás és a háttérkiszolgáló webes API-t az Azure AD.
3. Azure Redis Cache példányt létrehozni.
4. Az Alkalmazásbeállítások konfigurálása, és hozzon létre egy helyi adatbázist.
5. Futtassa az alkalmazást, és regisztráljon egy új bérlőt.
6. Alkalmazás-szerepkörök hozzáadása a felhasználók számára.

## <a name="prerequisites"></a>Előfeltételek
-   [Visual Studio 2017][VS2017]
-   [A Microsoft Azure](https://azure.microsoft.com) fiók

## <a name="create-the-tailspin-tenant"></a>A Dejójáték bérlő létrehozása

Dejójáték fiktív cég, amelyen a felmérések alkalmazás. Dejójáték használja az Azure AD és az alkalmazás regisztrálásához más bérlők engedélyezéséhez. Ezek az ügyfelek használhatja az Azure AD hitelesítő adatait az alkalmazás-ba való bejelentkezéshez.

Ebben a lépésben létre fog hozni egy Azure AD-címtár Dejójáték számára.

1. Jelentkezzen be az [Azure Portalra][portal].

2. Kattintson a **új** > **biztonság + identitás szakaszában** > **az Azure Active Directory**.

3. Adja meg `Tailspin` a szervezet neve, és adja meg a tartomány nevét. A tartománynév fog rendelkezni az űrlap `xxxx.onmicrosoft.com` és egyedinek kell lenniük. 

    ![](./images/running-the-app/new-tenant.png)

4. Kattintson a **Create** (Létrehozás) gombra. Az új könyvtár létrehozása néhány percig is eltarthat.

A végpont forgatókönyv végrehajtásához egy második Azure AD-címtár képviseli, amelyet az alkalmazás előfizet lesz szüksége. Az alapértelmezett Azure AD-címtár (nem Dejójáték) használja, vagy hozzon létre egy új könyvtárat erre a célra. A példákban a Contoso a fiktív ügyfélként használjuk.

## <a name="register-the-surveys-web-api"></a>Regisztrálja a felmérések webes API-k 

1. Az a [Azure-portálon][portal], kiválasztja a fiókját a portál jobb felső sarkában az új Dejójáték könyvtár váltani.

2. A bal oldali navigációs ablaktábláján válassza **Azure Active Directory**. 

3. Kattintson a **App regisztrációk** > **új alkalmazás regisztrációja**.

4. Az a **létrehozása** panelen adja meg a következő adatokat:

   - **Név**: `Surveys.WebAPI`

   - **Az alkalmazástípus**: `Web app / API`

   - **Bejelentkezés URL-cím**: `https://localhost:44301/`
   
   ![](./images/running-the-app/register-web-api.png) 

5. Kattintson a **Create** (Létrehozás) gombra.

6. Az a **App regisztrációk** panelen, jelölje be az új **Surveys.WebAPI** alkalmazás.
 
7. Kattintson a **Tulajdonságok** elemre.

8. Az a **App ID URI** szerkesztési mezőhöz, adja meg `https://<domain>/surveys.webapi`, ahol `<domain>` annak a könyvtárnak a tartománynév. Például:`https://tailspin.onmicrosoft.com/surveys.webapi`

    ![Beállítások](./images/running-the-app/settings.png)

9. Állítsa be **Multi-központjaként** való **Igen**.

10. Kattintson a **Save** (Mentés) gombra.

## <a name="register-the-surveys-web-app"></a>A felmérések webes alkalmazás regisztrálása 

1. Lépjen vissza a **App regisztrációk** panelen, majd kattintson **új alkalmazás regisztrációja**.

2. Az a **létrehozása** panelen adja meg a következő adatokat:

   - **Név**: `Surveys`
   - **Az alkalmazástípus**: `Web app / API`
   - **Bejelentkezés URL-cím**: `https://localhost:44300/`
   
   Figyelje meg, hogy a bejelentkezési URL-cím rendelkezik-e a port számát a `Surveys.WebAPI` alkalmazást az előző lépésben.

3. Kattintson a **Create** (Létrehozás) gombra.
 
4. Az a **App regisztrációk** panelen, jelölje be az új **felmérések** alkalmazás.
 
5. Másolja át az azonosítót. Ezt később lesz szüksége.

    ![](./images/running-the-app/application-id.png)

6. Kattintson a **Tulajdonságok** elemre.

7. Az a **App ID URI** szerkesztési mezőhöz, adja meg `https://<domain>/surveys`, ahol `<domain>` annak a könyvtárnak a tartománynév. 

    ![Beállítások](./images/running-the-app/settings.png)

8. Állítsa be **Multi-központjaként** való **Igen**.

9. Kattintson a **Save** (Mentés) gombra.

10. Az a **beállítások** panelen kattintson a **válasz URL-címek**.
 
11. Adja hozzá a következő válasz URL-CÍMEN: `https://localhost:44300/signin-oidc`.

12. Kattintson a **Save** (Mentés) gombra.

13. A **API-hozzáférés**, kattintson a **kulcsok**.

14. Adja meg a leírását, például `client secret`.

15. Az a **válasszon időtartama** legördülő menüben válasszon ki **1 év**. 

16. Kattintson a **Save** (Mentés) gombra. A kulcs mentésekor jön létre.

17. Mielőtt ezt a panelt elhagyni, másolja a kulcsnak az értéke.

    > [!NOTE] 
    > A kulcs nem fog megjelenni újra után elnavigál a panelt. 

18. A **API-hozzáférés**, kattintson a **szükséges engedélyek**.

19. Kattintson a **hozzáadása** > **API kiválasztása**.

20. A keresőmezőbe, keresse meg a `Surveys.WebAPI`.

    ![Engedélyek](./images/running-the-app/permissions.png)

21. Válassza ki `Surveys.WebAPI` kattintson **válasszon**.

22. A **delegált engedélyek**, ellenőrizze **hozzáférés Surveys.WebAPI**.

    ![Engedélyek delegálás beállítása](./images/running-the-app/delegated-permissions.png)

23. Kattintson a **válasszon** > **végzett**.


## <a name="update-the-application-manifests"></a>Az alkalmazásjegyzékeknek frissítése

1. Lépjen vissza a **beállítások** panelen a `Surveys.WebAPI` alkalmazást.

2. Kattintson a **Manifest** > **szerkesztése**.

    ![](./images/running-the-app/manifest.png)
 
3. Adja hozzá a következő JSON a `appRoles` elemet. Új GUID azonosítóját készítése a `id` tulajdonságok.

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

4. Az a `knownClientApplications` tulajdonság, vegye fel a felmérések webes alkalmazás, amely korábban a felmérések alkalmazás regisztrálásakor kapott Alkalmazásazonosító. Példa:

   ```json
   "knownClientApplications": ["be2cea23-aa0e-4e98-8b21-2963d494912e"],
   ```

   Ezt a beállítást a felmérések alkalmazás hozzáadása a webes API hívásához jogosult ügyfelek listáját.

5. Kattintson a **Save** (Mentés) gombra.

Most ismételje meg a felmérések alkalmazást ugyanezekkel a lépésekkel kivéve ne adjon hozzá egy bejegyzést a `knownClientApplications`. Használja az ugyanazon szerepkör-definíciók, de új GUID azonosítók előállításához.

## <a name="create-a-new-redis-cache-instance"></a>Hozzon létre egy új Redis Cache példányt

A felmérések alkalmazás Redis gyorsítótár OAuth 2 jogkivonatot használja. A gyorsítótár létrehozásához:

1.  Ugrás a [Azure Portal](https://portal.azure.com) kattintson **új** > **adatbázisok** > **Redis Cache**.

2.  Töltse ki a szükséges információt, beleértve a DNS-név, az erőforráscsoport, a hely, IP-címek. Hozzon létre egy új erőforráscsoportot, vagy használjon egy meglévő erőforráscsoportot.

3. Kattintson a **Create** (Létrehozás) gombra.

4. A Resis gyorsítótár létrehozása után nyissa meg az erőforrás a portálon.

5. Kattintson a **hívóbetűk** és másolja az elsődleges kulcsot.

A Redis gyorsítótár létrehozásával kapcsolatos további információkért lásd: [használata Azure Redis Cache hogyan](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache).

## <a name="set-application-secrets"></a>Set alkalmazás titkos kulcsok

1.  Nyissa meg a Tailspin.Surveys megoldást a Visual Studióban.

2.  A Megoldáskezelőben kattintson a jobb gombbal a Tailspin.Surveys.Web projektet, és válassza ki **felhasználói kulcsok kezelése**.

3.  A secrets.json fájl illessze be az alábbiakat:
    
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
   
    Cserélje le a következőképpen csúcsos zárójelek megjelenő elemek:

    - `AzureAd:ClientId`: A felmérések alkalmazás az alkalmazás azonosítója.
    - `AzureAd:ClientSecret`: A kulcs az Azure ad-ben a felmérések alkalmazás regisztrálásakor létrehozott.
    - `AzureAd:WebApiResourceId`: A App ID URI Azure AD-ben a Surveys.WebAPI alkalmazás létrehozásakor megadott. Az űrlap rendelkeznie kell `https://<directory>.onmicrosoft.com/surveys.webapi`
    - `Redis:Configuration`: Hozhat létre. Ez a karakterlánc a DNS neve alapján a Redis gyorsítótárt és az elsődleges elérési kulcsot. For example, "tailspin.redis.cache.windows.net,password=2h5tBxxx,ssl=true".

4.  Mentse a frissített secrets.json fájlt.

5.  Ismételje meg ezeket a lépéseket a Tailspin.Surveys.WebAPI projekt esetében, de illessze be a következő secrets.json. Cserélje le azt korábban a csúcsos zárójelek elemeket.

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

Ebben a lépésben szüksége lesz az Entity Framework 7 LocalDB használata egy helyi SQL-adatbázis létrehozásához.

1.  Nyisson meg egy parancsablakot

2.  Nyissa meg a Tailspin.Surveys.Data projekt.

3.  Futtassa az alábbi parancsot:

    ```
    dotnet ef database update --startup-project ..\Tailspin.Surveys.Web
    ```
    
## <a name="run-the-application"></a>Az alkalmazás futtatása

Futtassa az alkalmazást, indítsa el a Tailspin.Surveys.Web és a Tailspin.Surveys.WebAPI projektek.

A Visual Studio segítségével mindkét projekt futtatásához automatikusan F5, a következőképpen állíthatja be:

1.  A Megoldáskezelőben kattintson a jobb gombbal a megoldás, és kattintson a **indítási projektek beállítása**.
2.  Válassza ki **több kezdőprojekt**.
3.  Állítsa be **művelet** = **Start** Tailspin.Surveys.Web és Tailspin.Surveys.WebAPI projektek.

## <a name="sign-up-a-new-tenant"></a>Iratkozzon fel egy új bérlőt

Az alkalmazás indításakor nem jelentkezett be, így a kezdőlapon:

![Kezdőlap](./images/running-the-app/screenshot1.png)

A szervezet feliratkozáshoz:

1. Kattintson a **regisztrálja a vállalat Dejójáték**.
2. Jelentkezzen be az Azure AD-címtárral, amely a szervezet a felmérések alkalmazással jelöli. Be kell jelentkeznie rendszergazda felhasználóként.
3. Fogadja el a jóváhagyási kérése.

Az alkalmazás a bérlő regisztrálja, és majd kijelentkezik meg. Az alkalmazás kijelentkezik, mert később szüksége lesz a szerepkörökhöz beállítása az Azure AD, az alkalmazás használata előtt.

![Regisztráció után](./images/running-the-app/screenshot2.png)

## <a name="assign-application-roles"></a>Alkalmazás-szerepkörök hozzárendelése

Amikor a bérlő előfizet, AD a bérlői rendszergazda felhasználók szerepkörökhöz kell rendelni.


1. Az a [Azure-portálon][portal], az Azure AD-címtár, amellyel iratkozzon fel a felmérések app váltani. 

2. A bal oldali navigációs ablaktábláján válassza **Azure Active Directory**. 

3. Kattintson a **vállalati alkalmazások** > **összes alkalmazás**. A portál felsorolja `Survey` és `Survey.WebAPI`. Ha nem, akkor győződjön meg arról, hogy a regisztrációs folyamat befejeződött-e.

4.  Kattintson az felmérések alkalmazásra.

5.  Kattintson a **felhasználók és csoportok**.

4.  Kattintson a **felhasználó hozzáadása**.

5.  Ha a prémium szintű Azure AD, kattintson **felhasználók és csoportok**. Ellenkező esetben kattintson a **felhasználók**. (Az Azure AD Premium szerepkört rendel egy csoport szükséges.)

6. Jelöljön ki egy vagy több felhasználót, majd **válasszon**.

    ![Felhasználó vagy csoport kiválasztása](./images/running-the-app/select-user-or-group.png)

6.  Válassza ki a szerepkört, és kattintson a **válasszon**.

    ![Felhasználó vagy csoport kiválasztása](./images/running-the-app/select-role.png)

7.  Kattintson a **Hozzárendelés** gombra.

Ismételje meg a azonos Survey.WebAPI alkalmazásához szerepköröket hozzárendelni.

> Fontos: A felhasználó mindig meg kell adni a felmérést és a Survey.WebAPI azonos szerepkörök. Ellenkező esetben a felhasználónak kell inkonzisztens engedélyek, a webes API 403-as (tiltott) hibákhoz vezethetnek.

Most lépjen vissza az alkalmazást, és jelentkezzen be újra. Kattintson a **a felmérések**. Ha a felhasználó a SurveyAdmin vagy SurveyCreator szerepkör van hozzárendelve, megjelenik egy **felmérés létrehozása** gomb, amely azt jelzi, hogy a felhasználó rendelkezik-e egy új felmérés létrehozásához szükséges engedélyek.

![A felmérések](./images/running-the-app/screenshot3.png)


<!-- links -->

[portal]: https://portal.azure.com
[VS2017]: https://www.visualstudio.com/vs/
