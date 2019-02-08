---
title: Az Azure-ban CAF:Identity vonatkozó eszközeivel
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Identitás vonatkozó eszközeivel az Azure-ban
author: BrianBlanchard
ms.openlocfilehash: 81b0fa9cfee597da98d8b983fb155eac82d97bf8
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899343"
---
# <a name="identity-baseline-tools-in-azure"></a>Identitás vonatkozó eszközeivel az Azure-ban

[Identitásra építve](overview.md) egyike a [öt oktatnak, felhőalapú Cégirányítási](../governance-disciplines.md). Ez fegyelem módját a házirendekben, amelyek a konzisztencia és a felhasználói identitásokat, függetlenül a felhőszolgáltató folytonosságának biztosítása üzemeltető összpontosít, az alkalmazást vagy munkaterhelést futtatják.

A következő eszközök útmutató a felderítési hibrid identitás a szerepel.

**Az Active Directory (helyszíni):** Active Directory egy az identitásszolgáltató a leggyakrabban használt vállalati tárolására, és a felhasználói hitelesítő adatok ellenőrzéséhez.

**Azure Active Directory:** A szolgáltatott szoftver (SaaS) az Active Directory, képes egy helyszíni Active Directoryval való összevonás egyenértékű.

**Az Active Directory (IaaS):** Az Azure-beli virtuális gépen futó Active Directory-alkalmazás egy példányát.

Identitás számítástechnikai biztonsági vezérlősíkjáért. Így a hitelesítés az a szervezet hozzáférés guard a felhőbe. Cégeknek szükségük van egy identitás vezérlősík erősíti a biztonsági állapotuk és biztonságban tartja a felhőalapú alkalmazások.

## <a name="cloud-authentication"></a>Felhőalapú hitelesítés

Az első szempont a szervezet számára, aki a saját alkalmazások a felhőbe helyezheti át a megfelelő hitelesítési módszer választása.

Ha ezt a módszert választja, az Azure AD felhasználói bejelentkezési folyamat kezeli. Közvetlen egyszeri bejelentkezés (SSO) szolgáltatással párosítva, felhasználók jelentkezhetnek be a felhőalkalmazások írja be újra a hitelesítő adatok nélkül. A felhőalapú hitelesítés a két lehetőség közül választhat:

**Az Azure AD a Jelszókivonat-szinkronizálás**: A legegyszerűbben a helyszíni címtárobjektumok az Azure AD-hitelesítés engedélyezése. Ez a módszer is használható egyik módszer egy biztonsági mentési feladatátvételi hitelesítési módszerként abban az esetben a helyszíni kiszolgáló leáll.

**Az Azure AD átmenő hitelesítés**: Egy állandó jelszó érvényesítése az Azure AD hitelesítési szolgáltatásokat biztosít a szoftver ügynököt futtat egy vagy több helyszíni kiszolgálók használatával.

> [!NOTE]
> Vállalatok egy biztonsági követelmény, hogy azonnal érvénybe lépteti a helyi felhasználói fiók állapotok, jelszóházirendek, és a bejelentkezési óra gondolja át az átmenő hitelesítést.

**Az összevont hitelesítés:**

Ha ezt a módszert választja, az Azure AD a hitelesítési folyamat egy külön megbízható hitelesítési rendszere, például a helyszíni Active Directory összevonási szolgáltatások (AD FS) vagy egy megbízható harmadik féltől származó összevonási szolgáltató, a jelszó érvényesítése továbbítja.

A cikk [a megfelelő hitelesítési módszer választása az Azure Active Directory](/azure/security/azure-ad-choose-authn) tartalmaz egy döntési fa segítséget nyújtanak a szervezet számára legjobb megoldást választhatja.

Az alábbi táblázat a natív eszközöket, amelyek segíthetnek a házirendek és a folyamatok, amelyek támogatják a cégirányítási fegyelem részletes.

<!-- markdownlint-disable MD033 -->

|Consideration|A Jelszókivonat-szinkronizálás és a közvetlen egyszeri bejelentkezés|Az átmenő hitelesítés + közvetlen egyszeri bejelentkezés|Összevonás az AD FS rendszerrel|
|:-----|:-----|:-----|:-----|
|Amikor megtörténik a hitelesítés?|A felhőben|Egy biztonságos jelszó ellenőrzési exchange helyszíni hitelesítési ügynök után a felhőben|Helyszíni követelmények|
|Mik azok a helyszíni kiszolgáló követelmények meghaladják a kiépítési rendszer: Azure AD Connect?|None|Egy kiszolgáló minden további hitelesítési ügynök|Két vagy több AD FS-kiszolgálók<br><br>A szegélyhálózat-alapú vagy szegélyhálózat (DMZ) hálózatban két vagy több WAP-kiszolgálók|
|Mik a helyszíni Internet követelményei és hálózatkezelési túl a kiépítési rendszer?|None|[Kimenő Internet-hozzáférés](/azure/active-directory/hybrid/how-to-connect-pta-quick-start) a kiszolgálókról a futtató hitelesítési ügynökök|[Bejövő Internet-hozzáférés](/windows-server/identity/ad-fs/overview/ad-fs-requirements) a WAP-kiszolgálókat a szegélyhálózaton<br><br>Bejövő hálózati hozzáférést AD FS-kiszolgálók a WAP-kiszolgálókat a szegélyhálózaton<br><br>Hálózati terheléselosztás|
|Van egy SSL-tanúsítványra vonatkozó követelménnyel?|Nem|Nem|Igen|
|Van egy állapotfigyelési megoldás?|Nem szükséges|Ügynök állapota által biztosított [Azure Active Directory felügyeleti központ](/azure/active-directory/hybrid/tshoot-connect-pass-through-authentication)|[Azure AD Connect Health](/azure/active-directory/hybrid/how-to-connect-health-adfs)|
|Felhasználók beszerzésének egyszeri bejelentkezés a felhőbeli erőforrásokhoz a vállalati hálózaton belüli tartományhoz csatlakoztatott eszközökről?|Igen, a [közvetlen egyszeri bejelentkezés](/azure/active-directory/hybrid/how-to-connect-sso)|Igen, a [közvetlen egyszeri bejelentkezés](/azure/active-directory/hybrid/how-to-connect-sso)|Igen|
|Milyen bejelentkezési típusok támogatottak?|UserPrincipalName + jelszó<br><br>Az integrált Windows-hitelesítés [közvetlen egyszeri bejelentkezés](/azure/active-directory/hybrid/how-to-connect-sso)<br><br>[Alternatív bejelentkezési Azonosítóval](/azure/active-directory/hybrid/how-to-connect-install-custom)|UserPrincipalName + jelszó<br><br>Az integrált Windows-hitelesítés [közvetlen egyszeri bejelentkezés](/azure/active-directory/hybrid/how-to-connect-sso)<br><br>[Alternatív bejelentkezési Azonosítóval](/azure/active-directory/hybrid/how-to-connect-pta-faq)|UserPrincipalName + jelszó<br><br>sAMAccountName és jelszó<br><br>Integrált Windows-hitelesítés<br><br>[Tanúsítvány és az intelligens kártyás hitelesítés](/windows-server/identity/ad-fs/operations/configure-user-certificate-authentication)<br><br>[Alternatív bejelentkezési Azonosítóval](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id)|
|A Windows Hello for Business támogatott?|[Kulcs megbízhatósági modell](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Az Intune-nal a tanúsítvány megbízhatósági modell](https://blogs.technet.microsoft.com/microscott/setting-up-windows-hello-for-business-with-intune/)|[Kulcs megbízhatósági modell](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Az Intune-nal a tanúsítvány megbízhatósági modell](https://blogs.technet.microsoft.com/microscott/setting-up-windows-hello-for-business-with-intune/)|[Kulcs megbízhatósági modell](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Tanúsítvány megbízhatósági modell](/windows/security/identity-protection/hello-for-business/hello-key-trust-adfs)|
|Mik azok a többtényezős hitelesítési beállítások?|[Azure MFA](/azure/multi-factor-authentication/)<br><br>[A feltételes hozzáférés * az egyéni vezérlők](/azure/active-directory/conditional-access/controls#custom-controls-1)|[Azure MFA](/azure/multi-factor-authentication/)<br><br>[A feltételes hozzáférés * az egyéni vezérlők](/azure/active-directory/conditional-access/controls#custom-controls-1)|[Azure MFA](/azure/multi-factor-authentication/)<br><br>[Az Azure MFA-kiszolgáló](/azure/active-directory/authentication/howto-mfaserver-deploy)<br><br>[Külső MFA](/windows-server/identity/ad-fs/operations/configure-additional-authentication-methods-for-ad-fs)<br><br>[A feltételes hozzáférés * az egyéni vezérlők](/azure/active-directory/conditional-access/controls#custom-controls-1)|
|Milyen felhasználói fiók állapotok támogatottak?|Letiltott fiókokat<br>(legfeljebb 30 perces késleltetés)|Letiltott fiókokat<br><br>A fiók zárolva<br><br>A fiók lejárt<br><br>A jelszó lejárt<br><br>Jelentkezzen be óra|Letiltott fiókokat<br><br>A fiók zárolva<br><br>A fiók lejárt<br><br>A jelszó lejárt<br><br>Jelentkezzen be óra|
|Mik a feltételes hozzáférési beállításokat?|[Az Azure AD feltételes hozzáférés](/azure/active-directory/active-directory-conditional-access-azure-portal)|[Az Azure AD feltételes hozzáférés](/azure/active-directory/active-directory-conditional-access-azure-portal)|[Az Azure AD feltételes hozzáférés](/azure/active-directory/active-directory-conditional-access-azure-portal)<br><br>[Az AD FS-jogcímszabályok](https://adfshelp.microsoft.com/AadTrustClaims/ClaimsGenerator)|
|Támogatott régebbi protokollokra blokkolja?|[Igen](/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)|[Igen](/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)|[Igen](/windows-server/identity/ad-fs/operations/access-control-policies-w2k12)|
|Testreszabhatja a embléma, a lemezkép és a leírás a bejelentkezési oldalakon?|[Igen, az Azure AD Premium szolgáltatással](/azure/active-directory/customize-branding)|[Igen, az Azure AD Premium szolgáltatással](/azure/active-directory/customize-branding)|[Igen](/azure/active-directory/connect/active-directory-aadconnect-federation-management#customlogo)|
|Milyen speciális forgatókönyvek támogatottak?|[Az intelligens jelszózárolás](/azure/active-directory/active-directory-secure-passwords)<br><br>[Kiszivárgott hitelesítő adatok jelentések](/azure/active-directory/active-directory-reporting-risk-events)|[Az intelligens jelszózárolás](/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication-smart-lockout)|Többhelyes közel valós idejű hitelesítő rendszerrel<br><br>[Az AD FS extranetes fiókzárolás](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-soft-lockout-protection)<br><br>[Integráció a külső identitáskezelési rendszerek](/azure/active-directory/connect/active-directory-aadconnect-federation-compatibility)|

<!-- markdownlint-enable MD033 -->

> [!NOTE]
> Egyéni vezérlők az Azure AD feltételes hozzáférés jelenleg nem támogatja az eszköz regisztrálása.

## <a name="next-steps"></a>További lépések

A [hibrid identitás digitális átalakítási keretrendszer](https://resources.office.com/ww-landing-M365E-EMS-IDAM-Hybrid-Identity-WhitePaper.html?LCID=EN-US) kombinációk és megoldásokat, kiválasztása, valamint integrálni az egyes összetevők számos ismerteti.

A [Azure AD Connect eszköz](https://aka.ms/aadconnectwiz) segít a helyszíni címtárak integrálása az Azure ad-ben.
